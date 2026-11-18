# Task Tracker — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the team. Replace every `{{…}}`, drop sections for features the brief turned off, and add the live links.

Live base: [{{BASE_NAME}}]({{URL_BASE}}). New work: the [New Task form]({{URL_FORM}}), an agent, or the grid. The ticket standard: `{{TICKET_STANDARD_LOCATION}}`.

## What this is

{{COMPANY}}'s one backlog. Every ticket, code or not, for every project, lives in the **Tasks** table. Projects and Tags slice it; nothing lives in a second list.

Find a ticket by its **Ticket ID** (`{{ID_PREFIX}}-1021`). Nobody types record ids.

## The lifecycle

```
Raw ──▶ Backlog ──▶ Todo ──▶ In Progress ──▶ QA ──▶ Done
 │         ▲                     │           │
 ▼         │                     ▼           │
Needs Grooming                Blocked        └─ back to In Progress if the check fails
(a question)                  (a reason)
                                              Won't Do (a reason), from anywhere
```

| Status         | Means                         | Who moves it here                     | What must be true                                              |
| -------------- | ----------------------------- | ------------------------------------- | -------------------------------------------------------------- |
| Raw            | captured                      | anyone, the form, agents, routines    | a title                                                        |
| Needs Grooming | a question is open            | whoever found the gap                 | the question is the last PM Notes line                         |
| Backlog        | ready to be picked            | whoever groomed it                    | it passes the readiness checklist (ticket standard, section 4) |
| Todo           | committed: next               | **{{PRIORITISER}}**                   |                                                                |
| In Progress    | being worked                  | whoever picked it up, before starting |                                                                |
| QA             | finished, waiting for a check | whoever did the work                  | the evidence is the last PM Notes line: {{EVIDENCE_RULE}}      |
| Done           | accepted                      | **{{REVIEWER}}**                      |                                                                |
| Blocked        | waiting on something outside  | anyone                                | the reason and who unblocks it are the last PM Notes line      |
| Won't Do       | decided against               | a person                              | the reason is the last PM Notes line                           |

Agents and routines never set Todo, Done, or Won't Do.

## Your week

| When                     | What arrives                                                                                               | What you do                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| {{REVIEW_SCHEDULE}}      | `PM · Board Review` digest ({{DIGEST_TARGET}})                                                             | Read the first line, then **Do this today**. Those items are the ones only a person can clear.      |
| The same day, 20 minutes | —                                                                                                          | {{PRIORITISER}}: open **Ready**, move this week's tickets to Todo, assign them.                     |
| Every day, 5 minutes     | —                                                                                                          | {{REVIEWER}}: open **Review**, check the evidence, move each ticket to Done or back to In Progress. |
| Friday, 15 minutes       | —                                                                                                          | {{GROOMER}}: open **Intake** and **Needs Grooming**. Groom what you can; answer the questions.      |
| {{TRIAGE_SCHEDULE}}      | **Error triage only.** `PM · Error Triage` digest, only when there are new bugs ({{TRIAGE_DIGEST_TARGET}}) | Glance at the priorities. A Critical goes to Todo today.                                            |

## Filing a ticket

- **Through the form:** title, what you need and why, the project, a due date only if it is real. It lands in Raw; someone grooms it.
- **Through an agent:** say "make a ticket for …". The agent writes it to the ticket standard, sets the fields, and gives you the Ticket ID.
- **In the grid:** follow the ticket standard. Leave Dedupe Key and Board Flags empty.

A ticket someone can act on without asking you a question gets done sooner. That is the whole standard in one line.

## Working a ticket

The ticket standard, section 8, is the rule for everyone. In short:

1. Read the Description, then **PM Notes** (PM Notes wins), then the Dependencies. Do not start until every dependency is closed.
2. Set **In Progress** before you start.
3. Missing a fact? **Needs Grooming**, with the question in PM Notes. Waiting on someone outside? **Blocked**, with the reason.
4. Finished? The evidence in PM Notes{{#if CODE}}, the PR in PR Link{{/if}}, then **QA**.
5. Never delete a ticket. Close it.

**PM Notes** is append-only: read it, add one dated line at the end (`[YYYY-MM-DD] <you>: <what>`), save. Editing the field in the grid and replacing its text erases everyone's history.

## Epics

An epic is a ticket titled `Epic: …` whose Dependencies are its children. Its progress is in the **Epics** view (children closed out of children). Each child has the epic's tag and a key like `ONB-03` as its first line. The epic's first PM Notes line is the ticket map. When the digest says "all children closed", check the epic's acceptance and close it.

## Board Flags

`PM · Board Review` writes one line on every ticket that needs a person, and clears it when the ticket is fine again. The **Flagged** view lists them.

| Flag                                  | What to do                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------ |
| `stale intake`, `question unanswered` | Groom it, answer the question, or close it as Won't Do.                                    |
| `stale active`                        | Ask the assignee. Still going: add a PM Notes line. Stalled: back to Backlog, with a line. |
| `stale review`, `review, no evidence` | {{REVIEWER}}: check it, or send it back with what is missing.                              |
| `stale blocked`, `blocked, no reason` | Write the reason and who unblocks it, or chase the blocker.                                |
| `blocked, dependencies closed`        | The blockers are closed. Move it on if nothing else blocks it.                             |
| `started before …`                    | Work began before its dependencies closed. Check that is deliberate.                       |
| `depends on Won't Do …`               | A ticket it waits for was abandoned. Rethink this one, or remove the dependency.           |
| `closed with open dependencies …`     | Closed while something it waited for is open. Check one of them is not wrong.              |
| `epic, all children closed`           | Check the epic's acceptance and close it.                                                  |
| `epic, no children`                   | Write the children, or close it.                                                           |
| `not ready: …`                        | Fill what it names before anyone picks it up.                                              |
| `todo, unassigned`                    | {{PRIORITISER}}: assign it.                                                                |
| `effort off scale`                    | Re-estimate on the scale ({{EFFORT_SCALE}}). Over 13: split it.                            |
| `overdue`                             | Move the date, or do it.                                                                   |
| `assignee former`                     | Reassign it.                                                                               |
| `project not active`, `no project`    | Move it to an active project, or close it.                                                 |
| `dependency loop …`                   | Two tickets wait on each other. Remove one link.                                           |

You may clear a flag by hand; if the problem is still there, the next run writes it again.

## Changing things by hand

| You want to                   | Do this                                                                                                                                                              |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Add a project                 | New Projects row: name, area, owner, status Active{{#if DONE_ANNOUNCE}}, Announce Done if its finished work should be announced{{/if}}. No automation change needed. |
| Add a tag or a type           | Add the option in the field's settings. Tell whoever runs the routines; they never add options themselves.                                                           |
| Someone leaves                | Set their People row's Kind to Former. Never delete it. Board Review flags their open tickets.                                                                       |
| Pause a routine               | Mode `Paused` on its Tracker Settings row.                                                                                                                           |
| Change a threshold or the cap | Edit its Tracker Settings row. Read every run.                                                                                                                       |
| Change the ticket standard    | Edit `{{TICKET_STANDARD_LOCATION}}`, raise its Version, keep the end marker.                                                                                         |
| Reopen finished work          | A new ticket that names the old one. Never move a Done ticket back.                                                                                                  |

## What the routines never do

- Move a ticket to Todo, Done, or Won't Do, or move one backward.
- Change a ticket's description, priority, assignee, or project. (Error Triage sets them only on tickets it creates.)
- Delete a ticket, or create a project, tag, person, or select option.
- Write anything to the error tracker or the repository.
- Put a secret, an email address, or a customer's data into a ticket or a digest.

## When something looks wrong

| Symptom                                      | Look at                                                                                                                               |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| No digest this week                          | The routine's last run in the scheduler, then Last Run At and Last Run Summary on its settings row.                                   |
| Mode dropped to Dry Run on its own           | Kill Reason. A guard fired: read the counts, find the cause (a renamed status, a bulk edit that reset Status Changed At), raise Mode. |
| Every ticket looks blocked, or none is Ready | The Open Dependencies rollup's function (it must be SUM of Is Open).                                                                  |
| A flag keeps coming back                     | The flag is still true. Read its row in the table above.                                                                              |
| Two tickets for one production error         | Their Dedupe Keys. A hand-made ticket has none: copy the error-tracker key onto the one to keep and close the other as Won't Do.      |
| PM Notes history vanished                    | Someone replaced the field instead of appending. Restore it from the record's revision history.                                       |
