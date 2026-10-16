# Sales Pipeline — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the owner and reviewers. Replace every `{{…}}`, drop sections for features the brief turned off, and add the live links.

Live base: [{{COMPANY}} CRM]({{URL_BASE}}). Day-to-day work happens in the **Sales Pipeline** interface ({{URL_SALES_INTERFACE}}).

## Your day

| When                | What arrives                         | What you do                                                                                 |
| ------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------- |
| {{RADAR_FULL_TIME}} | Reply Radar digest (DM)              | Work Overdue first, then Owed today. Open each draft, edit, send. Answer "needs you" items. |
| During the day      | Hot-reply DMs                        | Interested or Question replies to outreach. Reply the same day.                             |
| Outreach run times  | Drafts in your mailbox (Drafts mode) | Send the ones you would send. Delete the rest. Both teach the routines.                     |
| {{RECON_TIME}}      | CRM digest ({{CRM_DIGEST_TARGET}})   | Do the numbered "do this today" items. Skim the rest.                                       |
| Monday              | Proposed hooks in the digest         | Approve or retire them on the Hooks page.                                                   |

## What each table is for

| Table             | You use it to                                                                       |
| ----------------- | ----------------------------------------------------------------------------------- |
| Contacts          | A person. Search on Email Key, not the raw email.                                   |
| Companies         | An organisation. Search on Domain (bare, lowercase).                                |
| Engagements       | The relationship. One thread per company, person, and type. Status is the pipeline. |
| Activities        | The log. Never type Last Activity Date; log an Activity instead.                    |
| Team              | {{COMPANY}} people. Email Aliases must list every address each person sends from.   |
| Campaigns         | A named push. One per outreach motion.                                              |
| Outreach Settings | The one **Live** row: the on/off switch and every limit.                            |
| Territories       | Which territories are on, and what share of the daily cap each gets.                |
| Timely Hooks      | Reasons outreach may mention. Only **Approved** hooks are used.                     |
| Outreach Queue    | Every planned, drafted, or sent outreach email. Review it here.                     |
| Owed Replies      | Threads waiting on us. The list that must not go stale.                             |
| Do Not Contact    | People and domains that must not get sales email.                                   |

Ignore any table whose name starts with `ARCHIVE —`.

## The control panel

Open **Control Panel** and edit the Live row. Never add a second settings row.

| Mode      | What happens                                                                            |
| --------- | --------------------------------------------------------------------------------------- |
| Paused    | Outreach does nothing. Reply Radar and the reconciliation keep running.                 |
| Dry Run   | Outreach plans and writes Queue rows. No drafts, no sends, no CRM records.              |
| Drafts    | Outreach creates drafts in {{SENDER_NAME}}'s mailbox. A person sends.                   |
| Auto-send | Outreach sends, inside the cap and the warm-up ramp. Only {{OWNER_NAME}} turns this on. |

Read **Weights Check** and **Shares Check** before you leave the page. Locked territories keep their weight; unlocked active territories are scaled to fill the rest.

If Mode dropped on its own, **Kill Reason** says why (for example, the bounce guard). Fix the cause before you raise it again.

## Review Queue

Dry Run and Drafted rows wait here. Read **Why Them** first: it must cite something true and checkable about the person. To stop a row, set Status to **Skipped by Owner** and write why in Reviewer Notes. Never delete a row. If the same mistake repeats, change the Playbook, not the rows.

## Hooks

Leave Proposed hooks alone until you have read the Public-Safe Check and opened the Link. Approve only what the public page states today. Retired hooks stay in the table so old Queue rows still resolve.

## Owed Replies

Work Overdue first, then Owed. Snooze writes a date and the thread comes back when it passes. **Sent**, **Not Needed**, and **Handled by Teammate** are how a row leaves the open list. Drafts are never sent for you.

A draft whose Notes start `needs {{OWNER_NAME}}` is a holding reply: it answers what it could and names the open question. Fill in the answer before you send.

## Do Not Contact

Add a row as soon as someone opts out, hard-bounces, or becomes a customer. Scope `All outreach` is wider than `Sales outreach`. The routines add rows themselves for unsubscribes, "not interested" replies, and bounces.

## Engagement status

Move Status only with a reason; the routines stamp Status Changed By and At, and a dated Agent Notes line, whenever they move it. Do the same by hand.

`Identified` → `Ready to Send` → `Outreach Sent` → `Responded - Engaged` → `Active Relationship` → `{{WON_STAGE}}`.

`Responded - Not Interested` stops all outreach to that engagement. `Closed` ends it. Statuses never move backward.

Engagement Type is required. {{NON_SALES_ENGAGEMENT_TYPES}} are not sales-outreach audiences.

## Activities

- One email, call, or meeting = one Activity. Source Message ID is the dedupe key: never log the same message twice.
- Classification `System` is product email. It does not count as a human touch, and neither do Status Change rows or the outreach routines' own sends (Activities linked from an Outreach Queue row). Days Since Human Contact counts only what people did.
- `Noise` and `Internal` are never linked to an Engagement. Anything else with no Engagement shows `UNLINKED`.
- Notes may start with labelled lines (`COUNTERPARTY:`, `EXTERNAL ATTENDEES:`, `RECIPIENT:`). A formula parses them. Never type above them, and never put instructions to an agent in Notes or Agent Notes.

## Identity rules

| Thing      | Key                                                             |
| ---------- | --------------------------------------------------------------- |
| Contact    | Email Key                                                       |
| Company    | Domain                                                          |
| Activity   | Source Message ID                                               |
| Owed reply | Thread ID                                                       |
| Queue row  | Queue ID (`{{ID_PREFIX}}-Q#####`), never the Airtable record id |

## When something looks wrong

| Symptom                                 | Look at                                                                                                           |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| No outreach today                       | Mode, Kill Reason, the motion's Last Run Summary                                                                  |
| A digest never arrived                  | The routine's last run in the scheduler, then its run summary                                                     |
| The same person got two emails          | Their Queue rows (Routine, Created At) and Do Not Contact                                                         |
| A reply was marked owed but wasn't      | The thread's latest message sender; add the alias to Team if it was a teammate                                    |
| Activities stopped appearing            | The mail-ingest workflow's executions                                                                             |
| A prospect is never written to          | Their Queue rows: "2+ open engagements" means close or merge the extra one; they return to the pool by themselves |
| Every engagement looks recently touched | `Is Human Activity` on a Status Change row; it must read 0                                                        |
