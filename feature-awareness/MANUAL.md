# Feature Awareness — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past an authentication wall on the person's behalf.

## Access (before the build)

Every database credential is **read-only**. Never hand the agent an admin key, a service-role key, or an owner password.

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the base will live.
2. **Repository.** Give the build agent a checkout of the production branch. For the routine, add a read-only deploy key to the runtime environment (or skip it; the routine then skips tests and new surface).
3. **Production database.** Create a dedicated read-only role: `SELECT` on the product schema only, no access to auth or secrets schemas, a statement timeout (for example 30 seconds), and a connection limit of 2. Put its credentials in the connector settings or the runtime's secret store. If the only connector available is an admin one, connect it and set every write-capable tool to **always deny** (migrations, DDL, branches, function deploys).
4. **Dev or staging database.** The same, pointed at the database people test in.
5. **Email provider** (optional). A key that can list and read sent emails and nothing else, if the provider offers scoped keys. Otherwise deny every send, cancel, update, and delete tool by policy.
6. **Page analytics** (optional). A read-only reporting role.
7. **Chat.** Connect the chat tool. Note the report target.

## If there is no usage log (developer task)

The single best production source is the product's own record of which routes production hits. If it does not exist, a developer adds it:

- A daily rollup table: day, method, route template, status class, caller kind (real, staff, test, monitor), request count.
- Written by the API middleware. Route templates only (`/orders/:id`), never raw URLs, query strings, or user ids.
- Exclude or label health checks, uptime monitors, and the evidence routine's own calls.
- Grant the read-only role `SELECT` on it. Tell the agent when it has a week of data; it adds the usage step to the routine.

## After the base is built

1. **Rollup functions.** Open each rollup on Features and confirm the function: every count = SUM (of the 1/0 helpers); Last Evidence Change = MAX.
2. **Status Changed At.** On Paths, confirm it is a last-modified-time field that watches **only** Auto test, Dev status, and Prod status, in `{{TIME_ZONE}}`. If the agent could not create it, create it now.
3. **Time zones.** Confirm every date-time field shows `{{TIME_ZONE}}`, 24-hour. The API cannot change a zone after creation.
4. **Select hygiene.** Look for options with trailing spaces or near-duplicates in Role, Domain, Surface, and Kind.
5. **Human fields.** On Features, open Tested by a human, Verified by, Date verified, and Human notes, and put `Owned by {{SIGNOFF_OWNER}}. Agents never write this.` in each field's description.
6. **Views.** Create every view in `SCHEMA.md` → Views, with its filter and sort.

## Interface

1. Finish the **Feature Awareness** interface (<link>):
   - **Coverage overview:** the number tiles, the Status (auto) chart, the Features grid grouped by Domain, the Live row's Last Run At and Last Sources.
   - **Risk register:** Features (Risk register view) and Paths (High risk, not proven view).
   - **Sign-off queue:** the list, and a detail page that shows Summary, the coverage numbers, and the linked Paths with their evidence. Make **only** the four human fields editable on this page.
2. **Sign off button** (if the brief chose it). Add a button to the detail page that runs the Sign-off stamp automation.
3. Share the interface with the people the brief names. The sign-off page is editable for {{SIGNOFF_OWNER}} only.

## Automations

Turn each on only after a test proves it, then clear the test values.

1. **Sign-off stamp.** Click **Sign off** on a test feature in the interface as yourself: Date verified becomes today and Verified by becomes your name. Then have the agent write Evidence notes on a second test feature through the API: Date verified and Verified by must stay empty. The agent never writes the human fields itself, so this proves only that no API write can set off the stamp. If it fires, fix the trigger before go-live.
2. **Settings change.** Change Mode on the Live row: `{{OWNER_EMAIL}}` gets one email with the old and new value.

## Routine

1. **Environment.** One agent environment for this routine, holding the database credentials, the email key, and the deploy key. Do not share it with unrelated routines.
2. **Tool policies.** Confirm they match `KICKOFF.md` Phase 7: no Airtable deletes or schema changes, no database writes of any kind, no email sends, one chat destination.
3. **Enable** after the supervised runs in KICKOFF Phase 8 pass.

## The owner's switches

| Switch                               | Where                       | When                                                                         |
| ------------------------------------ | --------------------------- | ---------------------------------------------------------------------------- |
| Mode: Paused → Dry Run               | Evidence Settings, Live row | After the first evidence pass is reviewed                                    |
| Mode: Dry Run → Live                 | Live row                    | After a Dry Run proposes only changes you can explain                        |
| Mode back up after a guard           | Live row                    | After you have read Kill Reason and the change list, and fixed the cause     |
| Caps and scan day                    | Live row                    | Any time. The routine reads them every run.                                  |
| Lifecycle (Retired, Behind a switch) | Paths                       | When code is removed or a switch hides a path. The routine never changes it. |
| The four sign-off fields             | Sign-off queue              | Yours alone.                                                                 |

## Checks a person does once

- One Feature's record history after the first Live run shows no change to any of the four human fields.
- A second Live run with nothing new made zero writes to Paths and Features.
- The Coverage overview's tiles add up: exercised + partial + never + unknown in prod equals live paths.
- One `Never` path, picked at random: run its count yourself (or ask a developer) and get zero.
- One dev `Verified` path, picked at random: the rows the evidence cites exist and were not made by a seed script.
