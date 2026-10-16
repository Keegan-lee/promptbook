# Workflow: mail and calendar ingest

Every email and every calendar meeting with an external counterparty becomes **one Activity** in the CRM, with Classification `Unclassified`. The nightly reconciliation classifies and links them. This is deterministic work, so it runs in a workflow engine, not an agent.

Two workflows, plus one shared helper and one manual backfill:

| Workflow         | Trigger                                                             | Writes                                      |
| ---------------- | ------------------------------------------------------------------- | ------------------------------------------- |
| Mail Capture     | every 4 hours during the day [06:00, 11:00, 15:00, 20:00 home time] | Activities: Email Sent, Inbound Email       |
| Calendar Capture | nightly [21:00 home time]                                           | Activities: Meeting, Meeting Scheduled      |
| Token helper     | called by the others                                                | nothing; returns a short-lived access token |
| Mail Backfill    | manual, one date window per run                                     | Activities with Source = Backfill           |

## Auth

- **Least privilege.** Ingest needs **read-only** scopes and nothing else: mail read-only and calendar read-only.
- **Google Workspace:** a service account with domain-wide delegation, impersonating each mailbox in turn, one scope per token. Some workflow tools' built-in Google nodes request a fixed, broad scope list; with delegation that is all-or-nothing, and the token request fails (`unauthorized_client`). Sign the JWT yourself (RS256, `iss` = service account, `sub` = the mailbox, `scope` = one scope, `aud` = the token endpoint, `exp` = 1 hour) and exchange it at the token endpoint.
- **Keep the private key in exactly one place:** the token helper, as a credential, never pasted into node source in several workflows. Every other workflow calls the helper with `{mailbox, scope}` and gets back an access token. A key copied into several workflows is readable by anyone with workflow read access and is impossible to rotate cleanly.
- **Microsoft 365:** an app registration with `Mail.Read` and `Calendars.Read` application permissions, limited to the watched mailboxes by an application access policy.

## Config (one place)

- `mailboxes`: every watched mailbox (brief, Block E).
- `internal`: the internal domains **plus** every teammate address on a domain the company does not own (a personal address, a university address). Read this list from the CRM's Team → Email Aliases at the start of each run, so ingest, reconciliation, and Reply Radar always agree. A shared public domain is never internal as a whole; only the listed address is.
- `home_time_zone`: `{{TIME_ZONE}}`.

## Mail Capture

1. For each mailbox: get a token from the helper, then list messages with a rolling, overlapping window. The overlap makes runs idempotent without a cursor; the dedupe in step 4 removes repeats.
   - Gmail query: `newer_than:2d in:anywhere -in:spam -in:trash -in:draft -category:promotions -category:social -category:updates -category:forums`. Excluding drafts matters: `in:anywhere` includes drafts, and without `-in:draft` every unsent draft (including the routines' own outreach and reply drafts) is logged as an email sent, a phantom outbound touch that moves statuses and resets recency.
   - **Paginate** with the page token until the end. Without pagination, anything past the first page is silently dropped on busy days.
2. Fetch each message's metadata only (no bodies): `Message-ID`, `References`, `In-Reply-To`, `From`, `To`, `Cc`, `Bcc`, `Subject`, `Date`, `List-Unsubscribe`, `List-Id`, `Precedence`, `Auto-Submitted`, `Return-Path`, plus the provider's snippet and thread id. Batch the calls (for example 25 at a time with a short pause) to stay under rate limits.
3. **Hard filter**, in order. Drop a message when:
   1. its Message-ID is missing, or already seen in this run (one thread reaches several mailboxes);
   2. `List-Unsubscribe` or `List-Id` is present (mailing lists);
   3. `Precedence` is bulk, list, auto_reply, or junk;
   4. `Auto-Submitted` is present and not `no`;
   5. the sender matches no-reply patterns: no-reply, do-not-reply, notification(s)@, mailer, bounce, postmaster, unsubscribe, automated, alert(s)@, billing@, invoice(s)@, receipt(s)@, support@, calendar-notification;
   6. the subject matches: unsubscribe, out of office, automatic reply, delivery status notification, undeliverable, read receipt;
   7. it is a calendar notification (invitation, accepted, declined, appointment booked) sent by the calendar system: meetings come from Calendar Capture only;
   8. every participant is internal, **unless** it is an outbound bulk send (below).
4. **Dedupe:** read existing Source Message IDs from Activities **once per run** (date within the last 14 days), then diff in memory. A per-item search inside the loop can collapse items and silently lose records.
5. **Direction, actor, counterparty:**
   - Outbound if From is internal, otherwise Inbound.
   - Actor: outbound → the sender; inbound → the first internal address in To, else in Cc. Never infer the actor from which mailbox the message was found in: polling order is a race.
   - Counterparty: outbound → the first external To, else the first external participant; inbound → the sender. Domain = the part after `@`.
   - Date: the message's received time converted to the **home time zone**, then the date. A UTC date puts late-evening mail on the wrong day.
6. **Outbound bulk sends** (Bcc fan-out). An outbound message is a bulk send when Bcc is non-empty, To contains "undisclosed-recipients", or To and Cc are both empty. Write **one Activity per external Bcc recipient**, each with its own key and counterparty. If the Bcc header is missing (only the sender's own copy carries it), write one flagged row with counterparty `undisclosed-recipients`. Poll the sending mailbox **first**, so its copy (with the Bcc header) wins the in-run dedupe over copies in other mailboxes.
7. **Write** each new Activity:

   | Field             | Value                                                                             |
   | ----------------- | --------------------------------------------------------------------------------- |
   | Activity Type     | `Email Sent` (outbound) or `Inbound Email`                                        |
   | Direction         | `Outbound` / `Inbound`                                                            |
   | Date              | home-zone date                                                                    |
   | Source            | `Mail Ingest`                                                                     |
   | Classification    | `Unclassified`                                                                    |
   | Source Message ID | the Message-ID with angle brackets removed, trimmed; bulk rows `{id}#{recipient}` |
   | Thread ID         | the provider's thread id                                                          |
   | Notes             | below, starting at the first character                                            |

   ```
   SUBJECT: {subject or "(no subject)"}

   COUNTERPARTY: {counterparty email}

   SNIPPET: {snippet, at most 500 characters}

   [actorEmail={actor}] [domain={counterparty domain}]{bulk markers}
   ```

   Bulk markers: ` [blast=true] [blastMessageId={id}] [blastSize={n}]`, or ` [blast=true] [recipients=unrecoverable] [blastMessageId={id}]`.

   Write select values that already exist only. Do not typecast new options in a live workflow.

## Calendar Capture

1. For each calendar: list single events (recurring series expanded) from 3 days ago to 14 days ahead, paginated.
2. Keep events with at least one external attendee. Drop room and resource addresses, and declined events.
3. Actor: the organiser if internal, else the first internal attendee. Counterparty: the first external attendee; list all external attendees in Notes.
4. Key: `cal:{iCalUID or id}#{start date, home zone}`. Recurring occurrences share one iCalUID, so the date keeps each occurrence distinct.
5. Activity Type: `Meeting` if the event has ended, otherwise `Meeting Scheduled`. Direction blank. Source `Calendar`.
6. Notes:

   ```
   MEETING: {title or "(no title)"}

   EXTERNAL ATTENDEES: {a, b, c}

   {description, at most 500 characters}

   MEET LINK: {conference link, if any}

   [actorEmail={actor}] [domain={counterparty domain}] [calendarMatch=true]
   ```

7. **Lifecycle** (fixes for gaps found in production):
   - A row captured as `Meeting Scheduled` is updated to `Meeting` once the event has ended. Do not leave it scheduled forever because the key already exists.
   - A rescheduled event creates a new key. Before writing, look for an existing `Meeting Scheduled` row with the same iCalUID and a different date: update its Date and key instead of adding a second row.
   - A cancelled event: append `[cancelled=YYYY-MM-DD]` to the existing row's Notes. Never delete the row.

Meetings recorded from a video tool without a calendar match carry `[calendarMatch=false]` and an unverified guest name. The reconciliation never links on those alone.

## Mail Backfill (manual)

A fork of Mail Capture for loading history in chunks:

- An explicit window: `after:YYYY/MM/DD before:YYYY/MM/DD`, plus the same exclusions.
- Real pagination with a page cap. **Throw** when the cap is hit rather than silently truncating.
- Request the Bcc header and keep the bulk-send fan-out, or older bulk sends are lost as internal-only.
- Source = `Backfill` and a ` [backfillWindow={start}..{end}]` marker. The reconciliation never moves a status on backfilled rows.
- Dedupe against the window ±2 days.
- Workflow-tool code steps may time out (for example after 60 seconds). Fetch metadata in an HTTP step, not inside a long code loop.

## Errors

- Route every failure to an error workflow that writes one error record and notifies the owner. Do not let failed items "continue" silently: an item with no payload simply vanishes.
- One mailbox's token failure must not abort the other mailboxes. Handle it per mailbox and report it.
- Keep execution logs, but never log message bodies or tokens.

## Verify (first run)

- Run Mail Capture over 30 days on one mailbox. Open 10 Activities: Notes shape exact, the key without angle brackets, Actor marker correct, nothing internal-only, no drafts.
- Send one Bcc test to two external test addresses: two Activities with `#recipient` keys.
- Create, reschedule, then cancel one test meeting with an external test attendee: one row, updated twice, never duplicated.
