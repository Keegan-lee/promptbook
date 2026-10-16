# SPINE kit — schema

The field contract for the Data Room base, the settings table both routines read, the SPINE pointer record, and the folders where SPINE files live. Build it with `_shared/AIRTABLE.md`. Substitute the brief's vocabularies wherever a select says **(brief)**.

One base, `{{COMPANY}} Data Room`. It is the company's **index and memory of why**:

- **Articles**: one row per document or link. The file itself lives in the shared drive.
- **Decisions**: why the company is shaped this way, including what was rejected. The why, not the numbers.
- **Observations**: raw notes. Logging one costs nothing.
- **Board Minutes** (module): meetings and written resolutions.
- **Sections**: internal shelves for Articles.
- **Company State Settings**: one row per routine, the switches.

Canonical numbers live in the SPINE, never in this base. A Decision points at the document that enacted it and, if it must name a figure, says "see SPINE §n".

## Modules

| Module                    | Flag                  | Default                                     | Adds                                                                        |
| ------------------------- | --------------------- | ------------------------------------------- | --------------------------------------------------------------------------- |
| Board Minutes             | `BOARD_MINUTES`       | on for an incorporated company with a board | the Board Minutes table and its links                                       |
| Share packs               | `PACKS`               | off                                         | Articles → Pack, and one view per pack (counsel, investor diligence, board) |
| Drive Filing              | `DRIVE_FILING`        | on                                          | the `CS · Drive Filing` settings row                                        |
| Filing registers Articles | `REGISTER_CANDIDATES` | off                                         | Drive Filing may create Article rows at Status `not started`                |

## Reusing an existing data room

If the person already has a data-room base, **map onto it**. Rename nothing other automations or people use. Add only what is missing: Articles → State Role, Execution, and Registered By; the Observations Source option `Company State`; the Company State Settings table. Record the mapping in the brief's ID map. Never move existing records between tables.

## Build order

1. Sections
2. Articles
3. Board Minutes (module)
4. Decisions
5. Observations
6. The cross-links: Sections ↔ Articles, Articles ↔ Board Minutes, Articles ↔ Decisions, Board Minutes ↔ Decisions, Decisions ↔ Observations, Board Minutes → Supersedes (self)
7. Company State Settings, then its two rows
8. The pointer record and the protocol record (Phase 5 of the kickoff, after the first documents exist)

Dates are ISO `YYYY-MM-DD`. Date-time fields use `{{TIME_ZONE}}`, 24-hour. Set the zone when you create each field: the API cannot change it later.

## Vocabularies (brief)

Every select below is filled from the interview. Defaults are offered in brackets and are proven starting points. Show the person the final lists before you build: options are hard to rename later.

| Vocabulary          | Used by                     | Default                                                                                          |
| ------------------- | --------------------------- | ------------------------------------------------------------------------------------------------ |
| Article Categories  | Articles → Category         | The diligence shelf list below                                                                   |
| Sections            | Sections → Name             | Corporate, Legal, Finance, Product, Go-to-market, People, Operations, Company State              |
| Entities            | Articles → Entity           | every legal entity in scope, plus each sister entity the brief names                             |
| Product lines       | Articles → Product          | `Company-wide`, then one per product line                                                        |
| Decision domains    | Decisions → Domain          | Corporate, Product, GTM, Finance, People, Operations                                             |
| Deciders            | Decisions → Decided by      | each person who decides, plus `Board`                                                            |
| Signatories         | Board Minutes → Signatories | each director, officer, or shareholder who signs                                                 |
| Registries          | Board Minutes → Filed With  | each corporate registry the company files with, plus `None`                                      |
| Packs               | Articles → Pack             | the external shares the company assembles (for example `Counsel`, `Investor diligence`, `Board`) |
| Observation authors | Observations → Author       | each team member, plus `Agent`                                                                   |
| Observation sources | Observations → Source       | Customer, Partner, Team, Market, Platform, Investor / Funder, Company State                      |

**Article Categories default:** Company Formation Documents, Intellectual Property, Contracts & Agreements, Regulatory & Compliance, Product Overview, Architecture & Infrastructure, Data & Analytics, Security & Reliability, Development Process, Market Overview, Competitive Analysis, User Personas & Demand Validation, Go to Market Strategy, Operational KPIs, Growth Metrics, Engagement, Pilot Data, Historical Financials, Forecasts, Unit Economics, Fundraising Details, Founding Team, Advisors & Investors, Hiring Roadmap, Investor Materials, Supporting Materials, Reference.

## Sections

A shelf. Primary field **Name**.

| Field    | Type            |
| -------- | --------------- |
| Name     | single line     |
| Articles | link → Articles |

## Articles

One document or link. Primary field **Name**.

| Field             | Type                 | Notes                                                                                                                                 |
| ----------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Name              | single line          | The document's title as people say it.                                                                                                |
| Description       | long text            | What it is and why it matters, in two or three sentences. Never paste the document's content.                                         |
| Category          | single select        | (brief) The diligence shelf.                                                                                                          |
| Section           | link → Sections      | The internal shelf. Prefer one.                                                                                                       |
| Types             | multi select         | doc, pdf, sheet, slides, folder, link, image, video, table, text, number                                                              |
| Status            | single select        | not started, in progress, draft, review, done                                                                                         |
| Execution         | single select        | Signed, Unsigned draft, Not signable. Feeds the authority order: only `Signed` outranks the SPINE. A blank reads as `Unsigned draft`. |
| Entity            | single select        | (brief) Which legal entity it belongs to.                                                                                             |
| Product           | single select        | (brief) `Company-wide` or a product line.                                                                                             |
| {{#if PACKS}}Pack | multi select         | (brief) The external shares this row belongs to. Blank means internal only.{{/if}}                                                    |
| State Role        | single select        | SPINE Pointer, Agent Protocol, SPINE Extract. Blank for every other document. Exactly one row is `SPINE Pointer`.                     |
| Documents         | attachments          | The file, when it is not in the shared drive.                                                                                         |
| Drive Link        | url                  | The file in the shared drive. On the pointer row, the current SPINE.                                                                  |
| Link              | url                  | Any other location.                                                                                                                   |
| Board Minutes     | link → Board Minutes | Module.                                                                                                                               |
| Decisions         | link → Decisions     |                                                                                                                                       |
| Registered By     | single line          | Provenance stamp for rows an agent creates: `Created by <routine> on <YYYY-MM-DD> from <source>`. People leave it blank.              |
| Last Update       | last modified time   |                                                                                                                                       |

A `done` row has at least one of Documents, Drive Link, or Link.

## Board Minutes (module)

One meeting or written resolution. Primary field **Name**.

| Field                | Type                 | Notes                                                                                                                                                                            |
| -------------------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name                 | single line          |                                                                                                                                                                                  |
| Date                 | date-time            | When it happened.                                                                                                                                                                |
| Effective Date       | date-time            | When it takes effect, if different.                                                                                                                                              |
| Type                 | single select        | Annual Meeting, Special Meeting, Written Resolution, Organizing Resolution, Directors' Resolution, Shareholders' Resolution, Shareholder Agreement, Reconstruction, Ratification |
| Status               | single select        | Scaffold, Draft, For Signature, Executed, Filed, Original Not Located, Reconstructed, Superseded. Only `Executed` and `Filed` count as enacted.                                  |
| Agenda & Resolutions | long text            |                                                                                                                                                                                  |
| Document             | link → Articles      | Prefer one. The Article that holds the signed file.                                                                                                                              |
| Signatories          | multi select         | (brief)                                                                                                                                                                          |
| Reference            | single line          | A filing or minute number.                                                                                                                                                       |
| Filed With           | multi select         | (brief)                                                                                                                                                                          |
| Securities Issued    | long text            | What was issued, to whom, as the minute states it. The cap table itself lives in the SPINE or its extract, not here.                                                             |
| Supersedes           | link → Board Minutes | Prefer one. The minute this one replaces. Name the inverse `Superseded By`.                                                                                                      |
| Decisions            | link → Decisions     | Each decision this minute enacted.                                                                                                                                               |

## Decisions

Table description: "Why the company is shaped this way, including what was rejected. Canonical numbers live in the SPINE; this table holds the why." Primary field **Decision**.

| Field                 | Type                 | Notes                                                                                       |
| --------------------- | -------------------- | ------------------------------------------------------------------------------------------- |
| Decision              | single line          | Stated as a resolution.                                                                     |
| Reasoning             | long text            | Why, in plain terms.                                                                        |
| Rejected alternatives | long text            | What else was considered and why it lost. The field that stops the question being reopened. |
| Status                | single select        | Locked, Provisional, Superseded, Reversed                                                   |
| Date decided          | date                 |                                                                                             |
| Decided by            | multi select         | (brief)                                                                                     |
| Domain                | single select        | (brief)                                                                                     |
| Documents             | link → Articles      | The document that enacted or records it.                                                    |
| Board Minute          | link → Board Minutes | Module. The minute that passed it.                                                          |
| Observations          | link → Observations  | The raw notes that led here. The inverse of Observations → Promoted to.                     |

No field for valuations, ownership figures, or amounts. **Only people create Decisions.** No routine ever creates or edits one: a proposal is not a decision.

## Observations

Table description: "Raw capture. Logging costs nothing. One paragraph, Author, Source, stop." Primary field **Observation**. **Exactly these five fields.** Do not add a tag, a score, a status, or a project.

| Field       | Type             | Notes                                                                |
| ----------- | ---------------- | -------------------------------------------------------------------- |
| Observation | long text        | Plain text. Agent records start with a fixed first line (below).     |
| Logged      | date-time        | `{{TIME_ZONE}}`, 24-hour.                                            |
| Author      | single select    | (brief) Team members plus `Agent`.                                   |
| Source      | single select    | (brief) Includes `Company State` for the sync's records.             |
| Promoted to | link → Decisions | Set only when this became a decision. Leave the original text alone. |

Two kinds of agent record, both Author `Agent`, Source `Company State`:

| Record         | First line                                                  | Written by        | Purpose                                                                                                                                                                                 |
| -------------- | ----------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reconcile item | `RECONCILE — §<n> — <one-line question>`                    | `CS · SPINE Sync` | One per open conflict. The SPINE's `[UNRECONCILED …]` marker cites its record id. The owner settles it by recording a Decision and setting Promoted to, or by fixing the source system. |
| Audit record   | `SPINE Sync — <YYYY-MM-DD> — <N> applied, <M> unreconciled` | `CS · SPINE Sync` | One per run that changed the SPINE (or one per Dry Run). The rollback instruction.                                                                                                      |

Agent records never edit an earlier Observation, human or agent.

## Company State Settings

One row per routine. Every routine reads its own row before it does anything. A person changes behaviour by editing the row, never the prompt. Primary field **Job**.

| Field             | Type          | Notes                                                                                                            |
| ----------------- | ------------- | ---------------------------------------------------------------------------------------------------------------- |
| Job               | single line   | `CS · SPINE Sync` or `CS · Drive Filing`.                                                                        |
| Mode              | single select | Paused, Dry Run, Live. Only the owner raises it. A guard may lower it and must write Kill Reason.                |
| Kill Reason       | long text     | Dated line from the guard that lowered Mode.                                                                     |
| Word Ceiling      | number        | SPINE Sync. `{{WORD_CEILING \| 2500}}`.                                                                          |
| Gather Floor Days | number        | SPINE Sync. `{{GATHER_FLOOR_DAYS \| 7}}`. The window is never shorter than this.                                 |
| Max Extracts      | number        | SPINE Sync. `{{MAX_EXTRACTS_PER_RUN \| 3}}` new extraction documents per run.                                    |
| Batch Limit       | number        | Drive Filing. `{{FILING_BATCH \| 50}}` files moved per run.                                                      |
| Standing Facts    | long text     | Used only when the brief keeps standing facts here instead of in the SPINE. Owner-only. One dated line per fact. |
| Last Run At       | date-time     |                                                                                                                  |
| Last Run Summary  | long text     | Counts and record ids only. Newest line on top. Keep the last 20 lines.                                          |
| Current Version   | single line   | SPINE Sync. The SPINE file name the pointer names after the last run, for example `SPINE_2026-03-02`.            |
| Failure Streak    | number        | SPINE Sync. Consecutive runs whose write failed. Reset to 0 on success.                                          |
| Notes             | long text     |                                                                                                                  |

Seed one row per routine the brief turns on, Mode = `Paused`. What each mode does:

| Job                 | Dry Run                                                                                                                                                                                                                               | Live                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `CS · SPINE Sync`   | Gathers, classifies, composes. Writes one audit Observation marked `DRY RUN` holding every proposed change and the full composed text. Creates no file, moves nothing, leaves the pointer, creates no reconcile items. DMs the owner. | Full run: reconcile items, new SPINE, read-back, archive, pointer, audit record, DM.                         |
| `CS · Drive Filing` | Plans every move and rename. Moves nothing. Report lists each planned move.                                                                                                                                                           | Moves and (if the brief allows) renames within the batch limit. Registers Articles only if the module is on. |

## The pointer record

The one Articles row every agent starts from. It is how anyone finds the current SPINE without knowing a file id.

| Field       | Value                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------ |
| Name        | `SPINE — Canonical Company State (READ FIRST) — v<YYYY-MM-DD>`                                                     |
| State Role  | `SPINE Pointer`                                                                                                    |
| Drive Link  | The current SPINE file's URL.                                                                                      |
| Description | `Version <YYYY-MM-DD> · <words> words · Last verified <YYYY-MM-DD>. Read this first. Authority order: <one line>.` |
| Category    | `Reference`                                                                                                        |
| Section     | `Company State`                                                                                                    |
| Status      | `done`                                                                                                             |
| Execution   | `Not signable`                                                                                                     |

Rules:

- **Exactly one** row has State Role `SPINE Pointer`. Its record id is permanent and goes into every kit's brief. Never create a second pointer; update this one.
- Only `CS · SPINE Sync` and the owner write it. The sync writes it **last** in every run, so a failure earlier leaves the old pointer valid.
- The Agent Operating Protocol is a second row with State Role `Agent Protocol`, cited from SPINE §0.
- Every document the sync extracts from the SPINE is a row with State Role `SPINE Extract`, Status `review` until the owner confirms it.

## Where SPINE files live

Three folders in the shared drive, created in the kickoff's Phase 4.

| Folder                                                    | Holds                                                  | Rule                                                                                             |
| --------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| `{{SPINE_FOLDER \| Company State/SPINE}}`                 | **Exactly one** SPINE file: the one the pointer names. | A second file there is an orphan from a failed run. The next sync resolves it (see the routine). |
| `{{SPINE_ARCHIVE_FOLDER \| Company State/SPINE Archive}}` | Every prior version, under its original name.          | Never trashed, never edited, never read by other agents.                                         |
| `{{EXTRACT_FOLDER \| Company State/Extracts}}`            | Documents the sync extracted from the SPINE.           | Each one is registered as an Article. The sync never edits one after creating it.                |

- **File name:** `{{SPINE_FILE_PREFIX | SPINE_}}<YYYY-MM-DD>`. A second version on the same day gets `_r2`, `_r3`. The date is the Last verified date.
- **Format:** `{{SPINE_FORMAT | a native document in the shared drive}}`, written as Markdown text. Keep it free of emoji and under 40,000 characters (`_shared/ROUTINES.md`, "Reading binding documents").
- **Never overwrite a SPINE in place.** Every version is a new file. The prior one moves to the archive.
- **Verify by content, never by size.** Some drives report a meaningless size for native documents (for example `1` byte for a complete file). A write is proven only by reading the content back and finding every section heading and the `END OF SPINE` line.
- These three folders are **protected** from Drive Filing. Add them to its protected list.
- Sharing: the SPINE folder is readable by every person and agent who should read the SPINE. The archive is readable by the owner. Neither is shared outside the company.

## Views

| Table        | View                                   | Filter / sort                                                                        |
| ------------ | -------------------------------------- | ------------------------------------------------------------------------------------ |
| Articles     | All Articles                           | none; sort Name                                                                      |
| Articles     | Draft / In Progress / In Review / Done | Status = each                                                                        |
| Articles     | Last Modified                          | sort Last Update descending                                                          |
| Articles     | Company State                          | State Role is not empty                                                              |
| Articles     | Agent Registered                       | Registered By is not empty; sort Last Update descending                              |
| Articles     | Signed                                 | Execution = Signed                                                                   |
| Articles     | {{#if PACKS}}One view per pack         | Pack has `<pack>`. A share link goes on this view only, never the whole base.{{/if}} |
| Decisions    | Locked                                 | Status = Locked; sort Date decided descending                                        |
| Decisions    | Provisional                            | Status = Provisional                                                                 |
| Observations | Human Log                              | Author is not `Agent`; sort Logged descending                                        |
| Observations | Open Reconciles                        | Observation starts with `RECONCILE —`, Promoted to is empty; sort Logged ascending   |
| Observations | Sync Log                               | Observation starts with `SPINE Sync —`; sort Logged descending                       |

`Open Reconciles` is a working list, not proof of resolution: the SPINE's markers are the truth. The sync reports any marker whose Observation is no longer in this view.

## Automations and interfaces

None are required. Do not add an importer that writes transcripts, tickets, or mail into Observations: they are meant to stay cheap and human. An optional interface page, **Company State**, shows the pointer row, the Agent Protocol row, the Open Reconciles view, and the two settings rows.

## ID map additions

Besides the usual base, table, field, and choice ids (`_shared/AIRTABLE.md`), record these. Every routine and every other kit is filled from them.

```
Pointer record: SPINE — Canonical Company State (READ FIRST) <record id>
Agent Protocol record: <name> <record id>
Settings rows: CS · SPINE Sync <record id> · CS · Drive Filing <record id>
Folders: SPINE <folder id> · SPINE Archive <folder id> · Extracts <folder id> · Inbox <folder id>
```
