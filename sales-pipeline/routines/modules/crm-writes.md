# MODULE: CRM WRITES — how the sales routines keep the CRM truthful

> Template module. The build agent appends it to every outreach routine and to Reply Radar, and fills every `{{…}}` from the brief and the ID map.

Base `{{CRM_BASE_ID}}`. Address every field **by id**. Before writing a select value, read the field's live choices. Never invent an option, and never pass `typecast: true` to an existing select. Never touch tables whose names start with `ARCHIVE —`. Never delete a record in any CRM table.

A formula field can come back as an error value instead of a value (for example Counterparty Email when Notes carry no address, or a date formula on a blank date). Treat any error as blank. Never match, exclude, or link on it.

{{FIELD_TRAPS}}

> Build agent: list this base's field-name traps from the ID map, one line each, naming the right id and the wrong one: two link fields to the same table, a dead legacy field with the same name as the live one, a synced field the API cannot write, a field whose name differs from this kit's. Write "none" if there are none. Traps like these have caused failed and misdirected writes.

## Who writes what

Routines that write the same records must not fight. This is the ownership table.

| Record                                                                                         | Sales routines (SP)                                                                                                                | Others (don't fight them)                                                                             |
| ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Outreach Queue, Owed Replies, Do Not Contact, Timely Hooks (Status → Proposed or Retired only) | **SP**                                                                                                                             | the owner edits them in the Control Panel                                                             |
| Engagement Status, Status Changed By/At, Agent Notes, Outreach Step, Next Touch Due            | **SP**, for its own sequences and replies                                                                                          | the nightly CRM Daily Reconciliation also moves Status on Activity evidence; both use the same ladder |
| Company / Contact / Engagement creation                                                        | **SP** in Drafts and Auto-send only, never in Dry Run                                                                              | the reconciliation creates them from inbound mail                                                     |
| Email Activities (sent and received mail)                                                      | **Not SP.** The mail-ingest workflow writes them. Exceptions: the 24-hour fallback below, and linking a Queue row's Sent Activity. | mail ingest                                                                                           |
| Status Change and Meeting Scheduled Activities                                                 | **SP** (Source = Agent)                                                                                                            | the reconciliation writes its own                                                                     |
| Classification and linking of ingested Activities                                              | not SP                                                                                                                             | the reconciliation                                                                                    |

## Natural keys and lookups

- **Contact:** Contacts.Email Key `{{FLD_CONTACT_EMAIL_KEY}}` (formula: lowercased, trimmed email). Search before creating. Write the address into Email `{{FLD_CONTACT_EMAIL}}`; the key derives itself.
- **Company:** Companies.Domain `{{FLD_COMPANY_DOMAIN}}`. Also search Company Name `{{FLD_COMPANY_NAME}}` before concluding a company does not exist; older companies may have no Domain.
  - Never derive a match domain from the Website field.
  - Free-mail and ISP domains are never company domains.
  - An individual with no organisation: the company is `{Full Name} (Individual)`.
  - A free-mail local part that plainly names an organisation (for example `oakstreetbakery@gmail.com`): create the Company under that trading name, leave Domain blank, and add "name needs verifying" to Notes.
  - Backfill Domain on any existing Company you touch when the business's own website establishes it.
- **Engagement:** {{IDS_ENGAGEMENT_FIELDS}}
- **Team:** {{IDS_TEAM_RECORDS}}
- **Campaigns:** {{IDS_CAMPAIGN_RECORDS}}

## Creating records (Drafts and Auto-send only; Dry Run creates nothing in the CRM)

Create only when a Queue row exists for the person, in this order:

1. **Company.** If no match by Domain or Name, create it with Company Name (the trading name), Domain, Website (the homepage), Email (only if the business published it), Territory Label (from Territories → CRM Territory Labels), Type (existing choices only), and Notes `Created by <routine> on <YYYY-MM-DD> from <source URL or tier>`.
2. **Contact.** Only when a named person is known. For an organisation's generic inbox, skip the Contact and keep the email on the Company and the Queue row. Fields: Name, Email, Company, and Tag (an existing choice, only when the evidence supports it).
3. **Engagement.** First check that the Company has no open Engagement (Status not Closed and not Responded - Not Interested).
   - **Two or more open Engagements: stop.** Do not guess. Mark the Queue row Cancelled with Reviewer Notes "2+ open engagements, needs owner", and list it in the summary. Silent mis-linking is worse than a skipped email: the wrong relationship would look warm and nobody would notice.
   - Exactly one open: use it.
   - None: create one with Company, Person (if a Contact exists), Engagement Type (from the audience map in the prospect pool), Status Identified, Owner (Assigned) `{{OWNER_NAME}}`, Campaign (this routine's), and Agent Notes `YYYY-MM-DD · <routine>: created for outreach (<tier>). Type inferred from <evidence>.`
4. Link the Queue row's Contact, Company, and Engagement to what you found or created.

## Status ladder (the same ladder the nightly reconciliation uses)

Forward: `Identified → Ready to Send → Outreach Sent → Responded - Engaged → Active Relationship → {{WON_STAGE | Onboarding Started}}`.

`Responded - Not Interested` and `Closed` are terminal side exits, allowed from any rung. **Never move an engagement backward.** Every rule sets a floor: if the engagement already sits at or above it, change nothing.

| Event                                                                   | Status floor                                                                                      |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Queue row Drafted (Drafts mode)                                         | Ready to Send                                                                                     |
| First routine email confirmed sent (Touch 1, Re-engage, Stalled Signup) | Outreach Sent                                                                                     |
| Human reply classed Interested, Question, Not now, or Referral          | Responded - Engaged                                                                               |
| Reply classed Not interested or Unsubscribe                             | Responded - Not Interested (terminal)                                                             |
| Meeting held                                                            | Active Relationship. Leave this to the reconciliation; sales routines only log Meeting Scheduled. |

Sales routines never set Closed or `{{WON_STAGE | Onboarding Started}}`; the reconciliation owns those. A bounce or an out-of-office reply never moves Status.

**Every status change needs all three of these.** Teams often share one login, so these records are the only audit trail:

1. **A Status Change Activity**: Activity Type "Status Change", Source "Agent", Date today, Engagement linked (Company and Contact when known), Classification "Internal" (a status change is bookkeeping, not contact), Notes `Status: <old> → <new>. Evidence: <Queue ID / Message-ID / thread>. By <routine>.`
2. **Status Changed By** = `{{OWNER_NAME}}`'s Team record (the routine acts on the owner's behalf), and **Status Changed At** = today.
3. **An Agent Notes line**, prepended so the newest stays on top: `YYYY-MM-DD · <routine>: <old> → <new> (<evidence>)`. Never delete earlier lines.

Also keep **Outreach Step** (the last touch number sent) and **Next Touch Due** (the date the next touch is due, blank when the sequence has ended) current on every send, reply, and stop.

## Email Activities: the ingest writes them; routines only fill gaps

- The mail-ingest workflow writes one Activity per real email. Its dedupe key is Source Message ID: the RFC 822 Message-ID **without** angle brackets.
- Always store that same value in the Queue's Message ID field. Read it from the sent message's `Message-Id` header (raw headers), not the provider's internal id.
- **Link every send to its Activity (every run).** For each of this routine's Sent rows from the last 14 days with an empty Sent Activity, search Activities for Source Message ID = the row's Message ID. Found: set the Queue row's Sent Activity. This is what keeps the routine's own email out of Is Human Activity, so it never resets Days Since Human Contact. The link is written on the Queue row; it changes nothing the reconciliation owns.
- **24-hour fallback.** If a Queue row was Sent more than 24 hours ago and no Activity carries its Message-ID, write exactly one Activity, then link it as Sent Activity: Activity Type "Email Sent", Direction "Outbound", Date = the send date, Source "Agent", Classification "Relationship", Source Message ID, Thread ID, Actor = the sender, and Contact, Company, Engagement, and Campaign linked. Notes must use the ingest's shape **exactly**, starting at the first character, because the Counterparty Email formula parses it:

  ```
  SUBJECT: <subject>
  COUNTERPARTY: <to email>
  SNIPPET: <first 200 characters of the body, one line>
  [actorEmail=<sender email>] [domain=<to domain>] [queue=<Queue ID>]
  ```

  Search Source Message ID first. Never create a second Activity for one Message-ID.

- **Never** classify, re-link, or delete Activities the ingest created. That is the reconciliation's job.

## Do Not Contact

Add a row whenever someone unsubscribes (any wording that asks us to stop), says they are not interested, hard-bounces (Reason Bounced, Email only, never the Domain), or the owner asks.

Fields: Entry (the email, or `@domain` for a domain-wide block), Email, Domain (only when the whole organisation asked), Reason, Scope ("Sales outreach" by default; "All outreach" if they asked never to be contacted), Added By "Routine", Source Thread ID, Notes (one line, no quotes).

Also apply the mail label `{{LABEL_DNC | Sales Pipeline/Do Not Contact}}` to the thread. Check for an existing row first; never duplicate.

## Writing style for every CRM text field

- One line, dated, factual. No quotes from email bodies, no personal details, no money, no confidential matter.
- Agent Notes and Queue fields are read by people. Write them the way the owner would want to read them a year from now.
