# Task Tracker — schema

The field contract. Build it with `_shared/AIRTABLE.md`. Substitute the brief's vocabularies wherever a select says **(brief)**. Rows and blocks marked **Code only** exist only when the brief says the team ships code from the tracker (KICKOFF Block D); rows marked **Announce only** only when it turned the Done announcement on (Block F). Drop the others.

One base, `{{BASE_NAME}}` (default: `<Company> ToDo`). Four tables:

- **Tasks**: every ticket the company runs, code and non-code, in one table. An epic is a ticket too.
- **Projects**: the products, programmes, and departments tickets belong to. Long-lived; a ticket has one.
- **People**: everyone who is assigned work, agents included. A directory of who does work, not a contact list.
- **Tracker Settings**: one row per scheduled job (`Board Review`, `Error Triage`). The switches and last-run summaries routines read and write.

**Status names are fixed after the build.** Formulas, views, automations, routines, and other kits test for them by name. The brief may rename them before the build (KICKOFF Block C); after that, a rename is a same-day change to every formula, view, automation, and routine prompt, so treat the names as permanent.

## Build order

1. People (scalar and select fields).
2. Projects (scalar and select fields), then Projects → Owner (link to People).
3. Tasks (scalar and select fields).
4. Tasks → Project and Tasks → Assignee. Use the inverses Airtable makes on Projects and People.
5. Tasks → Dependencies (a self-link; Airtable makes no inverse for it).
6. Tasks formulas `Is Open`, then the `Open Dependencies` rollup and the `Dependency Count` count, then `Ready`, `Ticket ID`, `Days In Status`, `Overdue`.
7. Tasks lookups (`Project Area`, and the code lookups when the brief has code).
8. Projects and People rollups (`Open Tasks`) and the Projects count.
9. Tracker Settings and its rows.

Dates are `YYYY-MM-DD`. Use a **date** field for due dates, not a date-time: a due date with a time zone attached drifts by a day for anyone outside that zone, and the reference build's date-times sat in a zone nobody worked in. Created and modified times show in `{{TIME_ZONE}}`, 24-hour. Set the zone when you create each field; the API cannot change it afterwards.

## Status

The default ladder. The **role** column is what routines and other kits key on; the name is what people see.

| Status         | Role       | Default meaning (the brief may change it)                                                                          | Who moves a ticket here                |
| -------------- | ---------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------- |
| Raw            | intake     | Captured, not yet looked at. Forms, agents, and routines create tickets here.                                      | anyone, any agent, any routine         |
| Needs Grooming | needs info | Someone looked. A business fact or a decision is missing; the question is the last line of PM Notes.               | a person, or an agent grooming it      |
| Backlog        | ready      | Groomed: the description passes the readiness checklist in `TICKET_TEMPLATE.md`. Not committed to.                 | a person, or an agent grooming it      |
| Todo           | committed  | A person decided it is next.                                                                                       | **{{PRIORITISER}} only**               |
| In Progress    | active     | Someone, a person or an agent, is working it now.                                                                  | whoever picks it up, before they start |
| QA             | review     | The work is finished and the evidence is in PM Notes. Waiting for a person to check it.                            | whoever did the work                   |
| Done           | done       | A person checked it and accepted it.                                                                               | **{{REVIEWER}} only** (see below)      |
| Blocked        | blocked    | Cannot move until something outside the ticket happens. The reason and who unblocks it are the last PM Notes line. | anyone                                 |
| Won't Do       | closed     | Decided not to do it. The reason is the last PM Notes line.                                                        | a person                               |

`Won't Do` is not in every tracker, and it earns its place. Closing abandoned work as `Done` counts it as shipped in every report and fires every "done" announcement. Keep the two apart.

Agents and routines never move a ticket to Todo, Done, or Won't Do. They never move one backward from QA or Done. The one exception is another kit's routine **recording a person's act it read from the source system**: the Blog kit's writer may set Done after it reads that a person published the post, because the publish was the acceptance. That kit's brief names the move; nothing else gets it.

## Tasks

Primary field **Title**.

| Field             | Type                | Written by               | Notes                                                                                                                                                                                        |
| ----------------- | ------------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Title             | single line         | anyone                   | Primary. At most 90 characters. Shapes in `TICKET_TEMPLATE.md` (`Bug: …`, `Epic: …`).                                                                                                        |
| ID                | autonumber          | —                        | Never reused.                                                                                                                                                                                |
| Ticket ID         | formula             | —                        | `"{{ID_PREFIX}}-" & {ID}`. People search and type this (`{{ID_PREFIX}}-1021`). Unpadded on purpose: a padded id breaks past its width, and people type the short form anyway.                |
| Status            | single select       | see Status               | The ladder above. Default for a new record: `Raw`.                                                                                                                                           |
| Priority          | single select       | anyone                   | Critical, High, Medium, Low. Meanings from the brief (Block C).                                                                                                                              |
| Type              | multiple select     | anyone                   | (brief) What kind of work. Default: Code, Bug, Feature, Build, Design, Research, Docs, Task, Outreach, Discussion, Epic. Areas and products are **not** types: they are Tags or the Project. |
| Tags              | multiple select     | people, build            | (brief) Epic tags (`<epic-slug>`) and area tags. Routines write only options that exist and never mint one.                                                                                  |
| Project           | link → Projects     | anyone                   | Exactly one. Prefer a single-record link.                                                                                                                                                    |
| Assignee          | link → People       | anyone                   | Usually one. A link, not a collaborator field, so an agent persona and a person without an Airtable seat can be assigned.                                                                    |
| Description       | rich text           | anyone                   | The brief: a self-contained prompt whoever picks the ticket up can execute cold. Shapes in `TICKET_TEMPLATE.md`.                                                                             |
| PM Notes          | rich text           | anyone                   | The decision log. Append-only, oldest first, one dated line per entry. **When PM Notes and Description disagree, PM Notes wins.** Grammar in `TICKET_TEMPLATE.md`.                           |
| Dependencies      | link → Tasks (self) | anyone                   | The tickets this one waits for. An epic's Dependencies are its children.                                                                                                                     |
| Due Date          | date                | anyone                   | Only when the date matters. Most tickets have none.                                                                                                                                          |
| Effort            | number, integer     | anyone                   | {{EFFORT_SCALE \| Fibonacci only: 1, 2, 3, 5, 8, 13. 13 means "split it".}} Off-scale values crept into the reference board (20, 47, 80); Board Review flags them.                           |
| Complexity        | single select       | anyone                   | trivial, standard, complex. Steers which engine or person takes it.                                                                                                                          |
| Source            | single select       | the creator              | manual, form, agent, routine, error-tracker, import. Who or what created the ticket. Set once.                                                                                               |
| Dedupe Key        | single line         | routines                 | The natural key of a ticket a routine created (`<tracker>:<issue id>`, `kw:<keyword>`). Plain text, exact match, one key per ticket. People leave it empty.                                  |
| External Ref      | single line         | anyone                   | The id of the same thing in another system: a CMS document, a CRM record, the old tracker's id after an import.                                                                              |
| Attachments       | attachments         | anyone                   | Screenshots and files. Never a secret.                                                                                                                                                       |
| Board Flags       | long text           | `PM · Board Review` only | One line, rewritten by the routine when it changes, empty when the ticket is clean: `YYYY-MM-DD · <flag>; <flag>`. A person may clear it; the next run writes it again if it is still true.  |
| Created           | created time        | —                        |                                                                                                                                                                                              |
| Last Modified     | last modified time  | —                        | All fields.                                                                                                                                                                                  |
| Status Changed At | last modified time  | —                        | Watches **Status only**. Drives staleness. If the API cannot create it, a person does (`MANUAL.md`).                                                                                         |
| Is Open           | formula             | —                        | Below. 1 or 0.                                                                                                                                                                               |
| Open Dependencies | rollup              | —                        | SUM of Dependencies → Is Open. Confirm the function in the UI.                                                                                                                               |
| Dependency Count  | count               | —                        | Of Dependencies.                                                                                                                                                                             |
| Ready             | formula             | —                        | Below. 1 when the ticket can be started today.                                                                                                                                               |
| Days In Status    | formula             | —                        | Below.                                                                                                                                                                                       |
| Overdue           | formula             | —                        | Below.                                                                                                                                                                                       |
| Project Area      | lookup              | —                        | Project → Area.                                                                                                                                                                              |

{{#if CODE}}
Engineering fields, when the brief says the team ships code from this tracker (Block D):

| Field          | Type          | Written by   | Notes                                                                                                                                     |
| -------------- | ------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Engine         | single select | anyone       | (brief) Who or what should execute it: `human` plus one option per agent executor the team runs. Pick it when assigning an agent persona. |
| Branch         | single line   | the worker   | The branch the work is on.                                                                                                                |
| PR Link        | url           | the worker   | Required before a code ticket moves to QA.                                                                                                |
| Error Link     | url           | Error Triage | The issue in the error tracker.                                                                                                           |
| Codebase       | lookup        | —            | Project → Codebase.                                                                                                                       |
| Repository URL | lookup        | —            | Project → Repository URL.                                                                                                                 |

Do not add fields for things nobody fills. The reference build carried a CI-status field and an agent-run-id field that stayed empty across 1,288 tickets; the PR carries that information already.
{{/if}}

{{#if DONE_ANNOUNCE}}
| Field | Type | Notes |
| ----------------- | ------ | ----------------------------------------------------------------------------------------------------------------------- |
| Project Announces | rollup | MAX of Project → Announce Done. 1 when the ticket's project announces finished work. The Done announcement watches it. |
{{/if}}

### Formulas

Field names below; the API stores ids, so translate as you build. Substitute renamed statuses.

```
Is Open
IF(OR({Status} = "Done", {Status} = "Won't Do"), 0, 1)

Ready
IF(AND(OR({Status} = "Backlog", {Status} = "Todo"), {Open Dependencies} = 0), 1, 0)

Days In Status
IF({Status Changed At}, DATETIME_DIFF(NOW(), {Status Changed At}, 'days'))

Overdue
IF(AND({Is Open} = 1, {Due Date}, IS_BEFORE({Due Date}, TODAY())), 1, 0)
```

- A dependency that was closed as `Won't Do` counts as closed, so `Ready` does not wait on it forever. `PM · Board Review` flags it, because the dependent ticket may no longer make sense.
- `NOW()` and `TODAY()` refresh on Airtable's schedule, not live. Views can use them. Routines compute ages themselves from `Status Changed At` and the run time in `{{TIME_ZONE}}`.

### Epics

An epic is a ticket with Type `Epic`, titled `Epic: <outcome>`. Its **Dependencies are its children**, so `Open Dependencies` is its open-children count and its `Ready` turns 1 only when every child is closed. Each child carries the epic's tag and a key (`<EPIC_KEY>-03`) in its first Description line. The epic's PM Notes holds the ticket map. `TICKET_TEMPLATE.md` → Epics has the shape and the two-pass create.

Airtable makes no inverse for a self-link, so a child cannot show its epic in a field. The tag and the key line are how a child points back. Do not add a second self-link for "parent": two links that mean the same thing drift apart.

## Projects

Primary field **Name**.

| Field          | Type          | Notes                                                                                                                                                                        |
| -------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name           | single line   | Primary. The name people use ("Mobile app", "Marketing", "Year-end close").                                                                                                  |
| Description    | rich text     | One paragraph: what it is and who owns it.                                                                                                                                   |
| Area           | single select | (brief) Product, Marketing, Sales, Finance, Admin by default. Looked up onto Tasks. Views slice by it.                                                                       |
| Status         | single select | Active, Paused, Done. Views hide tickets of projects that are not Active.                                                                                                    |
| Owner          | link → People | The person who answers questions about it.                                                                                                                                   |
| Tasks          | link → Tasks  | Inverse of Tasks → Project.                                                                                                                                                  |
| Task Count     | count         | Of Tasks.                                                                                                                                                                    |
| Open Tasks     | rollup        | SUM of Tasks → Is Open.                                                                                                                                                      |
| Start Date     | date          | Optional.                                                                                                                                                                    |
| End Date       | date          | Optional.                                                                                                                                                                    |
| Codebase       | single line   | **Code only.** Short name of the repository or app. Looked up onto Tasks.                                                                                                    |
| Repository URL | url           | **Code only.** Looked up onto Tasks.                                                                                                                                         |
| Announce Done  | checkbox      | **Announce only.** On: a ticket of this project reaching Done posts to the announcement channel. One automation serves every project; a new project needs no new automation. |

## People

Primary field **Name**.

| Field          | Type          | Notes                                                                                                                                                    |
| -------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name           | single line   | Primary. For an agent persona, the persona's name.                                                                                                       |
| Kind           | single select | (brief) Founder, Employee, Contractor, Advisor, Partner, Agent, Former. `Former` instead of deleting someone who left: their tickets keep their history. |
| Email          | email         | Work address. Optional for agents.                                                                                                                       |
| Chat ID        | single line   | The person's id in the chat tool, for mentions in digests.                                                                                               |
| Default Engine | single select | **Code only.** Agents only: the Engine option this persona runs on. Same options as Tasks → Engine.                                                      |
| Tasks          | link → Tasks  | Inverse of Tasks → Assignee.                                                                                                                             |
| Open Tasks     | rollup        | SUM of Tasks → Is Open.                                                                                                                                  |

Keep addresses, phone numbers, and social profiles out of this table. They belong in a CRM or an HR system with its own access rules. Every agent that reads the tracker would otherwise read them too.

## Tracker Settings

Primary field **Job**. One row per routine: `Board Review`, and `Error Triage` when the brief turns it on. Routines read their row at the start of every run.

| Field                   | Type          | Used by      | Notes                                                                                    |
| ----------------------- | ------------- | ------------ | ---------------------------------------------------------------------------------------- |
| Job                     | single line   | both         | Primary. `Board Review`, `Error Triage`.                                                 |
| Mode                    | single select | both         | Paused, Dry Run, Live. Created `Paused`. Only the owner raises it; a guard may lower it. |
| Kill Reason             | long text     | both         | Written by a guard that lowered Mode. The owner clears it.                               |
| Max New Tickets Per Run | number        | Error Triage | Default 8.                                                                               |
| Window Hours            | number        | Error Triage | Default 26: a daily run with two hours of overlap.                                       |
| Stale Active Days       | number        | Board Review | Default 10. In Progress longer than this is flagged.                                     |
| Stale Review Days       | number        | Board Review | Default 7. QA longer than this is flagged.                                               |
| Stale Intake Days       | number        | Board Review | Default 14. Raw longer than this is flagged.                                             |
| Stale Blocked Days      | number        | Board Review | Default 21. Blocked longer than this is flagged.                                         |
| Max Flag Changes        | number        | Board Review | Default 60. The mass-change guard.                                                       |
| Last Run At             | date-time     | both         | `{{TIME_ZONE}}`, 24-hour.                                                                |
| Last Run Summary        | long text     | both         | The run's report. Counts and Ticket IDs only.                                            |
| Last Changed By         | single line   | both         | The routine, or the person who last edited the row.                                      |

## Views

The API cannot create views. A person creates them from this list (`MANUAL.md`). Keep the list short: the reference board grew to 37 views, most of them one person's or one product's copy of another view. Add a view when a person or an automation needs it, and name the person who asked in its description.

| View             | Type     | Filter and sort                                                                                       | Notes                                                                    |
| ---------------- | -------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| All              | grid     | none. Grouped by Status.                                                                              | **Do not filter, do not delete.** Routines and imports assume it exists. |
| Intake           | grid     | Status = Raw. Created, oldest first.                                                                  | The grooming queue.                                                      |
| Needs Grooming   | grid     | Status = Needs Grooming. Priority.                                                                    | Questions waiting for a person.                                          |
| Ready            | grid     | Ready = 1. Priority, then Due Date, then Ticket ID.                                                   | What can be started today.                                               |
| In Flight        | kanban   | Status is In Progress, QA, or Blocked. Stacked on Status.                                             |                                                                          |
| Review           | grid     | Status = QA. Status Changed At, oldest first.                                                         | {{REVIEWER}}'s queue.                                                    |
| Blocked          | grid     | Status = Blocked. Days In Status, highest first.                                                      |                                                                          |
| Recently Done    | grid     | Status = Done, Status Changed At within the last 14 days.                                             | What shipped.                                                            |
| Epics            | grid     | Type has Epic, Is Open = 1. Fields: Ticket ID, Title, Dependency Count, Open Dependencies, Status.    | Progress = Dependency Count − Open Dependencies.                         |
| Flagged          | grid     | Board Flags is not empty. Priority.                                                                   | `PM · Board Review`'s findings.                                          |
| Due              | calendar | On Due Date. Is Open = 1.                                                                             |                                                                          |
| `<Person>` board | kanban   | Assignee has `<person>`, Is Open = 1 or Status Changed At within 7 days. Stacked on Status. Priority. | One per person who asks for one.                                         |
| Agent queue      | grid     | Assignee is an agent persona, Ready = 1. Priority.                                                    | What agents may pick up. One per agent persona when there are several.   |
| Last Modified    | grid     | none. Last Modified, newest first.                                                                    |                                                                          |
| New Task         | form     | —                                                                                                     | **Do not rename, do not delete** when an automation watches it.          |

Kanban views stack on Status. Hide the `Won't Do` and `Done` stacks on working boards.

## Automations

Create each as a draft, off. A person turns it on after a test record proves it (`MANUAL.md`).

| Name                 | Trigger                                                   | Action                                                                                                     | When                            |
| -------------------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------- |
| Form intake defaults | The New Task form is submitted                            | Update that record: Status `Raw`, Source `form`.                                                           | Always, if the form exists.     |
| Done announcement    | A record matches: Status = Done and Project Announces = 1 | Post to `{{ANNOUNCE_CHANNEL}}`: Ticket ID, Title, Project, Assignee. Copy the message text from the brief. | Only if the brief turned it on. |
| Settings change      | A Tracker Settings record's Mode changes                  | Email {{OWNER_EMAIL}} the Job, the old and new Mode, and Kill Reason.                                      | Optional.                       |

The reference build had one "done" automation per project, each with the project's record id inside its condition. Every new project meant a new automation, and a rebuilt base broke all of them. The `Announce Done` checkbox and the `Project Announces` rollup replace the record ids.

Recurring chores (a weekly "send the newsletter" ticket) are an optional scheduled automation that creates one Raw task. Leave it out unless the brief asks for it.

## What routines may write

| Writer                    | Tables and fields                                                                                                                                                                                                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `PM · Board Review`       | Tasks: Board Flags only. Tracker Settings: its own row's Last Run At, Last Run Summary, Last Changed By, Kill Reason; Mode only downward.                                                                                                                                                        |
| `PM · Error Triage`       | Tasks: create records with the fields in its template; on an existing ticket, PM Notes only (one appended line). Tracker Settings: its own row, the same fields as above.                                                                                                                        |
| Any other kit's routine   | Whatever that kit's template names, through the mapping in its brief (for example the Blog kit's existing-task-table model). Its status moves are an agent's (In Progress, QA, Blocked, Needs Grooming), plus Done only where it records a person's act it read (above). Never Todo or Won't Do. |
| An agent working a ticket | That ticket's Status (In Progress, QA, Blocked, Needs Grooming), PM Notes (append), Branch, PR Link. New tickets it was asked to create. `TICKET_TEMPLATE.md` → Working a ticket.                                                                                                                |
