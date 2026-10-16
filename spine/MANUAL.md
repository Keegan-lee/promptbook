# SPINE kit — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past an authentication wall on the person's behalf.

## Connections (before the build)

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the Data Room will live, and read access to the task tracker and CRM bases the brief names.
2. **Shared drive.** Connect the drive with access to the root the brief names. The account the routines use must be able to **move** files in it (on a shared drive, at least the content-manager level). It does not need delete rights; if the connector lets you, leave them out.
3. **Chat.** Connect the chat tool. Note the owner's DM target.
4. **System of record.** Create a **read-only** database role (or a read-only API key) for the product database, scoped to the tables and views the traction metrics need. Put the credential in the connector or the runtime's secret store, never in chat. Never hand the sync an admin key.
5. **Optional sources:** the accounting tool, conversation memory, any extra source in the brief. Read-only access only.

## After the base is built

1. **Time zones.** Confirm Observations → Logged and Board Minutes → Date and Effective Date use `{{TIME_ZONE}}`, 24-hour. The API cannot change a time zone after creation.
2. **Single links.** In the field editor, set these to allow one linked record: Articles → Section, Board Minutes → Document, Board Minutes → Supersedes.
3. **Descriptions.** Paste the table descriptions for Decisions and Observations from `SCHEMA.md`, and the field descriptions for Execution, State Role, Registered By, Rejected alternatives, and Promoted to. The API often drops them.
4. **Select hygiene.** Look for options with trailing spaces or near-duplicates. Fix them before anything is imported.
5. **Views.** Create every view in `SCHEMA.md` → Views, with its filter and sort. `Open Reconciles` and `Sync Log` are the owner's working views.
6. **Pack views (module).** If share packs are on, share each pack's **view** read-only from the UI, never the whole base. The owner does this click.
7. **Interface (optional).** Finish the Company State page: the pointer row, the protocol row, Open Reconciles, the two settings rows.
8. **Settings rows.** Confirm each row reads Mode = `Paused` and the limits match the brief.

## Drive

1. **Sharing.** The SPINE folder: readable by every person and agent who should read the SPINE. The SPINE Archive: the owner only (and the routine's account). The Extracts folder: as the SPINE folder. None of the three is shared outside the company.
2. **Inbox.** If Drive Filing is on, confirm everyone who drops files can add to the inbox folder, and that the routine's account can move files out of it.
3. **Executed documents.** If signed originals live in their own folder, confirm it is on the protected list in the brief.

## Routines

1. **Environment.** Put the read-only database credential in the routine environment's secret store. Do not share that environment with routines that do not need the database.
2. **Tool policies.** Confirm each routine's connector policies match its template's Install table. For both routines, every drive tool that trashes, deletes, copies, shares, or changes permissions is set to always deny, and every mail tool is denied.
3. **Enable in order**, each only after its stage in the kickoff's Phase 7: `CS · SPINE Sync`, then `CS · Drive Filing`.

## The owner's switches

These are the owner's alone. Nobody else moves them, and no routine moves them up.

| Switch                                                     | Where                                                           | When                                                                                |
| ---------------------------------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| SPINE Sync: Paused → Dry Run                               | Company State Settings                                          | After SPINE v1 and the protocol are approved and a few days of change have happened |
| SPINE Sync: Dry Run → Live                                 | Company State Settings                                          | After a dry run whose composed text the owner would accept as it stands             |
| Drive Filing: Paused → Dry Run → Live                      | Company State Settings                                          | After one dry-run plan the owner agrees with                                        |
| Word Ceiling, Gather Floor Days, Max Extracts, Batch Limit | Company State Settings                                          | Any time. Read on every run.                                                        |
| Standing facts                                             | SPINE §1 (or the settings row)                                  | Any time. Only the owner adds, edits, or removes one.                               |
| Settle a reconcile item                                    | Decisions + the Observation's Promoted to, or the source system | Whenever NEEDS YOU lists one                                                        |

## Checks a person does once

- Exactly one Article has State Role `SPINE Pointer`, and its Drive Link opens the file in the SPINE folder.
- The SPINE folder holds exactly one file.
- A dry-run audit record appears under Observations → Sync Log, and its composed text ends with `END OF SPINE`.
- After the first Live run, the prior SPINE sits in the archive and nothing was trashed.
- A test file dropped in the inbox is in the right folder after a Drive Filing Live run, and the inbox's hold folders are untouched.
