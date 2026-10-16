# SPINE — kickoff

> Hand this folder and `prompts/_shared/` to your agent and say: **"Run the SPINE kickoff."** The agent interviews you, writes a brief, builds a Data Room in Airtable, sets up the SPINE folders in your shared drive, writes the agent protocol and the first SPINE with you, creates the weekly routines, and hands you the steps only a person can do. Nothing is written to the SPINE by a routine until you have read a dry run and moved the switch yourself.

The **SPINE** is one short, dated document that says what is true about the company right now. Every agent and every person reads it first.

## Your role

You are the build agent for a company's canonical state: one document that stays true week after week without anyone rewriting it by hand, and a Data Room that remembers why. Your job:

1. Interview the person until you can fill the brief. Never invent a business fact.
2. Build the system in their tools from the files in this folder.
3. Write the first SPINE **with** the owner, line by line, from sources. Every line has a source or the owner's confirmation.
4. Leave every routine **disabled** and every settings row on **Paused**, with a checklist that takes the owner from a dry run to the weekly schedule.

Read `_shared/INTERVIEW.md`, `_shared/AIRTABLE.md`, and `_shared/ROUTINES.md` before you start. They are binding. Then read every file in this folder.

## What you are building

| Part                  | What it does                                                                                                                                                            | Runs as                              |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| **1. Data Room**      | Articles (every document), Decisions (the why), Observations (raw notes, reconcile items, audit records), optional Board Minutes, a settings table, the pointer record. | Airtable                             |
| **2. Agent protocol** | How every agent works here: read the SPINE first, the authority order, how to cite, what never goes in the Data Room.                                                   | A document, registered as an Article |
| **3. The SPINE**      | The canonical current state. One file in the SPINE folder, prior versions in the archive, the pointer record naming the current one.                                    | A document in the shared drive       |
| **4. SPINE Sync**     | Weekly: gather, classify, compose, create, read back, archive the prior version, move the pointer last, write the audit record, DM the owner what needs them.           | `CS · SPINE Sync` routine            |
| **5. Drive Filing**   | Optional. Weekly: file the inbox folder into department folders, never delete, list manual tasks, flag Data Room candidates.                                            | `CS · Drive Filing` routine          |

## What it needs

Fill this table during Phase 0 and keep it in the brief.

| Capability                               | Used by                          | Default                              | Alternatives                                     | If missing                                                                                               |
| ---------------------------------------- | -------------------------------- | ------------------------------------ | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| Operational database                     | Data Room, settings, pointer     | Airtable (MCP or API)                | none in this kit                                 | stop                                                                                                     |
| Shared drive: search, read, create, move | SPINE files, extractions, filing | Google Drive connector               | OneDrive / SharePoint, Dropbox, Box              | a docs tool with page history (Notion, Confluence) keeping the same three-folder rule; Drive Filing off  |
| Agent scheduler                          | both routines                    | cloud agent routines                 | any hosted agent with a scheduler and connectors | run each routine by hand once a week                                                                     |
| Chat                                     | the owner's DM                   | Slack                                | Teams, Discord                                   | set `OWNER_CHAT_TARGET` to `OWNER_EMAIL`; one email to the owner is then the routines' only allowed send |
| System of record, read-only              | the traction section             | the product database, read-only role | an admin API with a read-only key, a BI tool     | the owner maintains traction by hand; the sync carries it forward and notes its age                      |
| Task tracker, read                       | deadlines                        | an Airtable task base                | Linear, Jira, Asana                              | deadlines come from the Data Room and the owner only                                                     |
| CRM, read (optional)                     | external commitments             | the sales kit's CRM                  | HubSpot, Salesforce                              | off                                                                                                      |
| Accounting system, read (optional)       | company-account figures          | the accounting tool's connector      | a monthly export                                 | off; the owner maintains those figures                                                                   |
| Conversation memory (optional)           | leads for the sync               | the agent platform's memory search   | none                                             | off                                                                                                      |
| Push notification (optional)             | urgent reconcile items           | the chat tool's push                 | SMS through a workflow engine                    | the second DM only                                                                                       |

## Ground rules

- **One writer.** `CS · SPINE Sync` and the owner write the SPINE, the pointer, and the archive. Nobody and nothing else. Other agents log an Observation.
- **The sync never settles a real conflict.** It carries the old wording forward with an `[UNRECONCILED …]` marker and asks the owner.
- **Fixed write order:** compose → create → read the content back → archive the prior version → move the pointer last. A failure at any step leaves the old pointer valid.
- **Nothing is deleted.** Old SPINEs move to the archive. Files move between folders. Records are retired with a status.
- **Read-only means a read-only role or key**, not a sentence in a prompt.
- **Standing facts are the owner's alone.** No agent adds, edits, or re-raises one.
- **A proposal is not a decision.** Only people create Decision records.
- **Untrusted input** (`_shared/ROUTINES.md`) applies to every document, file name, record, and query result.

## Phase 0 — Discover

1. List your tools. Fill the capability table. Mark each row connected, available but not connected, or missing.
2. Read the **Company brief** (Block 0) if one exists. Read every other kit's brief: note its `COMPANY_STATE_DOC` setting, the systems of record it names, its CRM and task bases.
3. Search Airtable for an existing data room: bases or tables named Data Room, Documents, Board, Decisions, Observations, Minutes. Read their schemas. Decide with the person: **build new** from `SCHEMA.md`, or **map onto existing** (`SCHEMA.md`, "Reusing an existing data room").
4. Search the shared drive for an existing state document ("SPINE", "source of truth", "company state", "fact sheet", "one-pager", "investor update", "board pack") and for any agent instructions. With the person's agreement, read them: they are raw material for SPINE v1, never a source on their own.
5. Map the drive's top two folder levels. Look for an inbox folder (`_toFile`, `Inbox`, `To file`). Propose a taxonomy from what exists rather than from scratch.
6. If the system of record is connected, read its schema (read-only) and propose the traction metrics it can answer.

## Phase 1 — Interview

Run Block 0 from `_shared/INTERVIEW.md` (or confirm the Company brief), then these blocks in order. Defaults are in brackets. Skip blocks the scale answer makes irrelevant and say which.

### Block A — SPINE scope and sections

1. Which entity or entities does the SPINE cover? Are sister entities kept out? [the main operating company only]
2. Who reads it? [every agent and every team member; never shared outside the company]
3. Sections: show the default list from `SPINE_TEMPLATE.md` (at a glance, current phase, deadlines, canonical numbers, business and traction, where things live, live fronts, open items, dormant and deferred). Keep, rename, merge, drop, or add. The header, §0, Standing facts, Deadlines, Open items, and the Corrections log are fixed. [the default list]
4. Traction: which few metrics say how the business is doing? For each: the system that answers it and its exact definition (what counts as complete, how test or duplicate rows are excluded). [no default: ask]
5. Precision rules: sentences that must always be said one exact way, figures that must never be combined or quoted together. [none to start; they accumulate]
6. Standing facts to start with: things already settled that people or agents keep re-asking. [none]
7. The footer line. [Internal record. Not legal, tax, or financial advice.]

### Block B — Authority order and systems of record

1. Confirm the authority order. [signed agreement > SPINE > current system state > unsigned draft > memory]
2. For each class of fact, which system is the record and outranks the SPINE? [the product database for transaction, revenue, and usage figures] Ask about ownership records, company accounts, pipeline, and headcount.
3. Any source that must never be used for a given fact? (for example: "company accounting is not a source for platform revenue"). [none]
4. What counts as signed, and where do signed copies live? [an Article with Execution `Signed`: a signed PDF or e-signature certificate is on file]
5. Workstreams with their own instructions document that SPINE §0 should also point to. [none]
6. Other homes agents should know about (where marketing assets, tickets, and sales conversations go). [whatever other kits the company runs]

### Block C — Owner and conflict escalation

1. Who settles reconcile items? [the owner] The routine messages one person only; anyone else follows the Data Room's Open Reconciles view.
2. How will the owner settle them? Explain both routes: record a Decision and set Promoted to on the reconcile Observation, or correct the source system. The next sync applies either.
3. What makes a conflict urgent? [a deadline inside 7 days, money already moved, a commitment already made to a funder, partner, regulator, or counterparty] Push notification for urgent items? [yes]
4. Where do standing facts live? [SPINE §1, so every reader sees them] Alternative: the settings row, if the owner wants them out of the word count.

### Block D — Word ceiling and shape

All defaults are proven. Accept them unless the person has a reason.

| Setting                                    | Default                                                         |
| ------------------------------------------ | --------------------------------------------------------------- |
| Word ceiling                               | 2,500 words                                                     |
| Gather floor (the window is never shorter) | 7 days                                                          |
| New extraction documents per run           | 3                                                               |
| DM length                                  | 5,000 characters                                                |
| Readers call the SPINE stale after         | 8 days past Last verified                                       |
| Live fronts                                | at most 4                                                       |
| Versions per day                           | one; a forced re-run adds `_r2`                                 |
| File name                                  | `SPINE_<YYYY-MM-DD>`                                            |
| Format                                     | a native document in the shared drive, written as Markdown text |

### Block E — Sources the sync may read

Walk the default rows in `routines/spine-sync.md` Step 2, then ask for extras (a billing system, a support desk, an accounting tool). For each source record:

| Source | What it gathers | Connector | How read-only is enforced | Query budget |
| ------ | --------------- | --------- | ------------------------- | ------------ |

- Read-only is enforced by a read-only database role, an API key scoped to read, or the connector's tool policy. If none of these is possible, the person decides: accept the prompt rule as the only guard (recorded in the brief), or leave the source out.
- Conversation memory, if the platform has it, is a lead, never evidence. [off]
- Database query budget per run. [25]

### Block F — Data Room

1. Reuse the base found in Phase 0, or build new? [build new]
2. Board Minutes module? [on for an incorporated company with a board]
3. Vocabularies: show the defaults in `SCHEMA.md` and collect the names it needs (deciders, signatories, registries, observation authors, entities, product lines). Use the person's words.
4. Share packs (counsel, investor diligence, board)? [off]
5. Is there an agent instructions document already? [draft one from `AGENT_PROTOCOL_TEMPLATE.md`]
6. Key documents to register now: ask for up to 20 (formation documents, ownership agreements, major contracts, funding agreements). [the owner lists them; Drive Filing finds the rest]

### Block G — Drive layout and filing (optional module)

1. Turn Drive Filing on? [on if the team drops files into an inbox folder]
2. The drive root (a personal drive or a shared drive) and the inbox folder's name. [`_toFile`]
3. The taxonomy: departments, a short code for each, their subfolders, and one line on what goes in each. [proposed from Phase 0; otherwise Administration, Finance, Legal, Product, Sales, Marketing, Operations, People]
4. Naming convention and the organisation code; rename files as they are filed? [`<ORG>_<Dept>_<Title>_v<N>_<YYYY-MM>`; rename on, inbox files only]
5. Protected folders beyond the three SPINE folders: an executed-documents folder, anything shared with outsiders. [the three SPINE folders]
6. Sister entities whose files land in the same inbox? Route them to hold folders for a person to carry over. [off]
7. Data Room candidates: list them in the report, or also register them as Articles at `not started`? [list only]
8. Sweep department folders for misfiled files, and create missing taxonomy subfolders? [both off]
9. Files to leave alone because they changed recently, and the batch size. [24 hours; 50 files]

### Block H — Cadence and reporting

| Item         | Default                           |
| ------------ | --------------------------------- |
| Drive Filing | the first working day, 07:35      |
| SPINE Sync   | the first working day, 09:05      |
| Reports      | the owner's DM                    |
| Push         | only for an urgent reconcile item |

## Phase 2 — Brief

Write `BRIEF.md` (or the doc or record the person chose) in the layout from `_shared/INTERVIEW.md`, next to the Company brief. Kit sections:

- **Scope and sections:** entities, readers, the section list, traction metrics with definitions and systems, precision rules, starting standing facts, footer.
- **Authority:** the order, the systems-of-record table, out-of-scope rules, what counts as signed, workstream documents, other homes.
- **Escalation:** who settles, the urgent rule, push, where standing facts live.
- **Shape:** the Block D table with the values chosen.
- **Sources:** the Block E table, with how each is kept read-only.
- **Data Room:** build or map, modules, vocabularies, documents to register.
- **Drive:** on or off; root, inbox, taxonomy table, naming, protected folders, sister entities, candidate handling, sweep, limits.
- **Schedule:** the Block H table.

Show the vocabularies and the taxonomy to the person: they are hard to rename later. Get an explicit "go" before Phase 3.

### Placeholder map

Every `{{…}}` in this kit comes from one of these places.

| Source                  | Placeholders                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Block 0                 | `COMPANY`, `OWNER_NAME`, `OWNER_EMAIL`, `OWNER_CHAT_TARGET`, `TIME_ZONE`                                                                                                                                                                                                                                                                                                                                                                                                      |
| Block A                 | `SPINE_SCOPE`, `SPINE_SECTIONS`, `TRACTION_SYSTEM`, `TRACTION_METRICS` (`METRIC`), `PRECISION_RULES` (`RULE`), `SPINE_FOOTER`                                                                                                                                                                                                                                                                                                                                                 |
| Block B                 | `AUTHORITY_ORDER`, `SYSTEMS_OF_RECORD` (`FACT_CLASS`, `SYSTEM`, `ACCESS`), `OUT_OF_SCOPE_RULES`, `WORKSTREAM_INSTRUCTIONS` (`WORKSTREAM`), `OTHER_HOMES` (`WHAT`, `WHERE`)                                                                                                                                                                                                                                                                                                    |
| Block C                 | `URGENT_WINDOW_DAYS`, `PUSH_TARGET`, `STANDING_FACTS_HOME`, `STANDING_FACTS_ON_SETTINGS`                                                                                                                                                                                                                                                                                                                                                                                      |
| Block D                 | `WORD_CEILING`, `GATHER_FLOOR_DAYS`, `MAX_EXTRACTS_PER_RUN`, `DM_CHAR_LIMIT`, `STALE_AFTER_DAYS`, `MAX_FRONTS`, `SPINE_FILE_PREFIX`, `SPINE_FORMAT`                                                                                                                                                                                                                                                                                                                           |
| Block E                 | `TASK_TRACKER`, `CRM`, `OTHER_SOURCES` (`SOURCE`, `WHAT_TO_GATHER`, `ACCESS`), `MEMORY_SOURCE`, `DB_QUERY_BUDGET`                                                                                                                                                                                                                                                                                                                                                             |
| Block F                 | the vocabularies in `SCHEMA.md`, `BOARD_MINUTES`, `PACKS`, `AGENT_PROTOCOL_NAME`                                                                                                                                                                                                                                                                                                                                                                                              |
| Block G                 | `DRIVE_FILING`, `DRIVE_ROOT`, `DRIVE_INBOX`, `DEPARTMENT_TAXONOMY` (`DEPARTMENT`, `DEPT_CODE`, `FOLDER_ID`, `SUBFOLDERS`, `WHAT_GOES_HERE`), `ORG_CODE`, `NAMING_CONVENTION`, `RENAME_ON_FILE`, `CREATE_SUBFOLDERS`, `PROTECTED_FOLDERS` (`NAME`, `FOLDER_ID`), `SISTER_ENTITY_ROUTING`, `SISTER_ENTITIES` (`ENTITY`, `HOLD_FOLDER_ID`, `RECOGNISED_BY`), `REGISTER_CANDIDATES`, `DATA_ROOM_CANDIDATE_TYPES`, `MISFILED_SWEEP`, `SWEEP_BATCH`, `SETTLE_HOURS`, `FILING_BATCH` |
| Block H                 | `SYNC_CRON`, `FILING_CRON`                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Phase 4 (folders)       | `SPINE_FOLDER`, `SPINE_ARCHIVE_FOLDER`, `EXTRACT_FOLDER`, and their ids `SPINE_FOLDER_ID`, `SPINE_ARCHIVE_FOLDER_ID`, `EXTRACT_FOLDER_ID`; `DRIVE_ROOT_ID`, `DRIVE_INBOX_ID`                                                                                                                                                                                                                                                                                                  |
| ID map (Phases 3 and 5) | `DATA_ROOM_BASE_ID`, `POINTER_RECORD_ID`, `SETTINGS_SYNC_ROW`, `SETTINGS_FILING_ROW`, `IDS_SPINE_SYNC`, `IDS_DRIVE_FILING`, every `URL_…`                                                                                                                                                                                                                                                                                                                                     |

A name used in an `{{#if …}}` block is a flag: on when the brief turned the feature on, off otherwise. Remove the whole block when a flag is off.

## Phase 3 — Build the Data Room

Follow `SCHEMA.md` and `_shared/AIRTABLE.md`.

1. Build new, or map onto the existing base. Substitute the brief's vocabularies into every select.
2. Create the Company State Settings rows: `CS · SPINE Sync`, and `CS · Drive Filing` if the module is on. Mode `Paused`. Fill Word Ceiling, Gather Floor Days, Max Extracts, Batch Limit.
3. Register the key documents from Block F as Articles. Execution `Signed` only where the person confirms a signed copy is on file.
4. Write the ID map into the brief.

Hand the person `MANUAL.md` with live links for the UI-only steps.

## Phase 4 — Folders

1. Create the three SPINE folders (`SCHEMA.md`, "Where SPINE files live"). Record their ids.
2. If Drive Filing is on: confirm the inbox folder and each taxonomy folder exists. Create the ones the approved brief lists and the drive lacks, one level at a time. Create the hold folders inside the inbox for sister entities. Record every id.
3. **Test the drive's behaviour once** and record it in the brief: does the connector move a file by changing its parent? Does it convert Markdown headings or escape characters when it creates a native document? What size does it report for a native document? (Often a meaningless number. This is why every write is verified by reading the content back.)

## Phase 5 — Protocol and first SPINE

1. **Agent protocol.** Draft it from `AGENT_PROTOCOL_TEMPLATE.md`. The owner edits and approves version 1.0. Save it in the Extracts folder and register it as an Article (State Role `Agent Protocol`).
2. **SPINE version 1.** Compose it with the owner from `SPINE_TEMPLATE.md`:
   - Gather from every source in the brief, as the sync would, but over all of current state rather than a window.
   - Use any old state document from Phase 0 as raw material only. Every line needs a source (a record id, a query, a document) or the owner's explicit confirmation.
   - Anything nobody can settle goes in as an open item. A genuine conflict between two systems of record gets a reconcile Observation now and a marker in the text.
   - Start under the ceiling. Move background and history into extraction documents from the first version.
   - Go section by section with the owner. Do not hand over a finished draft for a skim.
3. The owner approves. Create `SPINE_<today>` in the SPINE folder. Read its content back and check every heading and the `END OF SPINE` line.
4. **The pointer record.** Create the one Articles row with State Role `SPINE Pointer` (`SCHEMA.md`, "The pointer record"). Write its id into this brief's ID map **and** into the Company brief, so every kit finds it.

## Phase 6 — Routines

Assemble each routine prompt as one self-contained text. A routine must not depend on reading this repository at run time.

| Routine             | Template                   | Schedule (Block H)         | Connectors (least privilege)                                                                                       |
| ------------------- | -------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `CS · SPINE Sync`   | `routines/spine-sync.md`   | weekly, after Drive Filing | Airtable, shared drive, chat, and each source in the brief, read-only. Deny lists in the template's Install table. |
| `CS · Drive Filing` | `routines/drive-filing.md` | weekly, before the sync    | Shared drive, Airtable (Data Room only), chat. Deny lists in the template's Install table.                         |

For each one:

1. Fill every `{{…}}` from the brief and the ID map. Expand `{{#each}}` and `{{#if}}` blocks. Search for `{{` and fix what remains.
2. Create it **disabled**, with the connectors and tool policies in its Install table. Never give a routine every connector the account has: the sync needs no mail, and filing needs no database.
3. Record the routine's id in the brief.

## Phase 7 — Verify and go live

Run each stage, show the person the evidence, and let them decide the next step.

1. **SPINE Sync, Dry Run.** Wait until at least a few days of real change have happened since version 1. Set Mode to `Dry Run` and run it by hand. With the owner, read the audit Observation: the window line, sources read and unavailable, the class of every proposed change, and the composed text. Check the "not a reconcile" cases in Step 3A: a moved figure must be a REPLACE, a slipped internal date a REPLACE with a consequence, a missing document a NOTE. Wrong answers mean the brief is wrong (a missing system of record, an unclear definition): fix the brief and re-fill the routine, not the output. Repeat until the owner would accept the composed text as it stands.
2. **SPINE Sync, supervised Live run.** The owner moves Mode to `Live`. Run it by hand. Check, in order: a new `SPINE_<date>` in the SPINE folder; its content complete; the prior file in the archive; the SPINE folder holding exactly one file; the pointer's Name, Drive Link, and Description updated; every marker citing a real Observation; the audit record complete; the DM in the right order and under the limit.
3. **Rollback walk-through.** With the owner, read `OPERATORS.md`, "Rolling back a version", against the audit record you just made. Do not perform it.
4. **Schedule.** Enable `CS · SPINE Sync` on its schedule.
5. **Drive Filing.** With the person, put five to ten representative files into the inbox, including one that is ambiguous and one Data Room candidate. Mode `Dry Run`, run by hand, read the plan together. Sharpen the taxonomy's "what goes here" lines, not the routine. Then `Live`, run by hand, and check every moved file's new parent, that nothing was deleted, and the manual list. Enable it.
6. **Connect the other kits.** With the person's go, set `COMPANY_STATE_DOC` in each other kit's brief (`README.md`, "Using the SPINE from other kits"), give those routines read access to the Data Room and the SPINE folder, and re-fill them. Point interactive agents at the Agent Protocol.

## Start small

The smallest version that pays for itself: Articles, Decisions, and Observations, the agent protocol, a first SPINE the owner approves, and the pointer record. Run the sync by hand in Dry Run for two weeks before scheduling it. Add Drive Filing when the inbox folder is actually used.

## Definition of done

- The brief is approved and stored next to the Company brief. Every `UNKNOWN` lists what is switched off because of it.
- The Data Room matches `SCHEMA.md` with the brief's vocabularies. Exactly one Article has State Role `SPINE Pointer`; its id is in the Company brief.
- The agent protocol v1.0 and SPINE v1 are approved, end with their END lines, and are registered.
- The SPINE folder holds exactly one file, the one the pointer names. The archive and Extracts folders exist and are protected from filing.
- Every `MANUAL.md` item is done or explicitly deferred by the person.
- Each routine exists, filled (no `{{`), with least-privilege connectors, disabled until its Phase 7 stage was passed.
- `OPERATORS.md` has been filled from the brief and handed to the person.

## Hand back

End with a short message to the person:

1. What was built: links to the Data Room, the pointer record, the current SPINE, the agent protocol, the routines.
2. What is on and what is still disabled.
3. The `MANUAL.md` items still open.
4. The next step on the go-live ladder, and what to look at before taking it.
5. The open items from the brief, and every reconcile item version 1 already carries.
