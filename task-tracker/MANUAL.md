# Task Tracker — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past an authentication wall on the person's behalf.

## Access (before the build)

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the base will live (or the existing base, if Phase 0 chose Adopt).
2. **Old tracker** (if importing). Export the open tickets to CSV, or connect a read-only connector. The agent never writes to the old tool.
3. **Chat.** Connect the chat tool. Note the digest target.
4. **Repository** (if code). Give the build agent a checkout. For the error triage, add a read-only deploy key to the routine's environment.
5. **Error tracker** (if error triage). A read-only connector or token scoped to the organisation. Where the connector cannot be scoped, every write tool is denied by policy (`KICKOFF.md` Phase 7).
6. **Platform health** (optional). A service account with viewer roles only, stored as a secret in the routine's environment. Never paste the key into chat or into the repository.

## After the base is built

1. **Status Changed At.** On Tasks, confirm a last-modified-time field that watches **only** Status, in `{{TIME_ZONE}}`. If the agent could not create it, create it now. Without it, Board Review cannot tell how long a ticket has sat anywhere.
2. **Rollup functions.** Open each and confirm: Tasks → Open Dependencies = SUM; Projects → Open Tasks = SUM; People → Open Tasks = SUM{{#if DONE_ANNOUNCE}}; Tasks → Project Announces = MAX{{/if}}. The API may not return the function, and a COUNTALL here makes every ticket look blocked.
3. **Time zones.** Every date-time field shows `{{TIME_ZONE}}`, 24-hour. Due Date is a date with no time.
4. **Defaults.** Tasks → Status defaults to `Raw` for a new record.
5. **Select hygiene.** Look for trailing spaces and near-duplicates in Type, Tags, and Area. Two options that mean the same thing (`Action` and `Action (do)`, `docs` and `Docs`) split every view that filters on one of them.
6. **Single links.** Tasks → Project: turn on "allow linking to a single record" if the agent could not.
7. **Owner-only fields.** In each field's description, write who owns it: Board Flags `Written by PM · Board Review. Clear it if you like; it comes back if still true.` · Dedupe Key `Written by routines. Leave empty.` · PM Notes `Append only, one dated line per entry. Wins over the Description.`

## Views

Create every view in `SCHEMA.md` → Views, with its filter, sort, and grouping, and the per-person boards the brief names:

- [ ] All (grouped by Status) · **do not filter, do not delete**
- [ ] Intake · Needs Grooming · Ready · In Flight (kanban) · Review · Blocked · Recently Done
- [ ] Epics · Flagged · Due (calendar) · Last Modified
- [ ] Agent queue (one per agent persona)
- [ ] `<Person>` board, one per person who asked: {{PERSON_BOARDS}}

Hide `Done` and `Won't Do` stacks on the kanban views.

## Form

1. Publish **New Task** (<link>). Visible fields: Title, Description, Project, Due Date. Description text: "Say what you need and why. Add links. A ticket someone can act on without asking you a question gets done sooner."
2. Decide who can open it (the team only, or anyone with the link).
3. **Do not rename or delete the form view** once the automation below watches it.

## Automations

Turn each on only after a test proves it, then delete the test record and say you did.

1. **Form intake defaults.** Submit the form once with a title starting `[TEST]`. The new record shows Status `Raw` and Source `form`.
   {{#if DONE_ANNOUNCE}}2. **Done announcement.** Reconnect the chat account inside the automation (never paste a webhook secret into chat or a ticket). In a test base, or on a `[TEST]` ticket in a project with Announce Done on, set Status to Done: one message arrives in `{{ANNOUNCE_CHANNEL}}`. Set it back. Never test it with a real ticket.{{/if}}
2. **Settings change** (optional). Change Mode on a settings row: `{{OWNER_EMAIL}}` gets one email.

## Repository (if code)

1. Review and merge the agent-pointer change to the repository's onboarding file (KICKOFF Phase 4.5), if you said yes to it.

## Routines

1. **Environments.** One agent environment per routine, or one shared by these two only. The error-triage environment holds the platform health secret; nothing else does.
2. **Tool policies.** Confirm they match `KICKOFF.md` Phase 7: Board Review cannot create or delete records; Error Triage cannot delete records or change anything in the error tracker; each posts to one destination.
3. **Enable** each after the supervised runs in KICKOFF Phase 8 pass.

## Cut-over (if importing)

1. On the agreed date, stop new work in the old tool: archive the board, or post its new home in its description.
2. Tell the team, once, where to file work now: the form, an agent, or the grid.

## The owner's switches

| Switch                             | Where                          | When                                                                          |
| ---------------------------------- | ------------------------------ | ----------------------------------------------------------------------------- |
| Mode: Paused → Dry Run → Live      | Tracker Settings, each row     | After reading a Dry Run you can explain                                       |
| Mode back up after a guard         | Tracker Settings               | After reading Kill Reason and fixing the cause                                |
| Stale thresholds, Max Flag Changes | Board Review row               | Any time. The routine reads them every run.                                   |
| Window Hours, Max New Tickets      | Error Triage row               | Any time.                                                                     |
| Announce Done                      | Projects, each row             | When a project should, or should no longer, announce finished work.           |
| Ticket standard                    | `{{TICKET_STANDARD_LOCATION}}` | Edit it, raise its Version, keep the end marker. Routines read it every run.  |
| Todo, Done, Won't Do               | Tasks                          | Yours ({{PRIORITISER}}, {{REVIEWER}}). No routine or agent makes these moves. |

## Checks a person does once

- Two tickets, A depending on B: A is not in the Ready view until B is closed.
- An epic's Dependency Count equals its number of children, and the Epics view shows the right progress.
- After the first Live Board Review: three flagged tickets, picked at random, really need what the flag says.
- A second Board Review with no board changes wrote no Board Flags changes.
  {{#if ERROR_TRIAGE}}- An error that already has an open ticket got a "still firing" line, not a second ticket.{{/if}}
