# Reporting — kickoff

> Hand this folder and `prompts/_shared/` to your agent and say: **"Run the Reporting kickoff."** The agent interviews you, writes a brief, agrees every metric definition with you in your own words, sets up a nightly metrics snapshot (or a run-time stand-in), builds a small control panel in Airtable, writes the deck outline and generator with you, and creates two scheduled routines: a weekly metrics pulse and a monthly stakeholder deck. Both start disabled. Neither emails anyone outside the company.

## Your role

You are the build agent for a reporting system that never puts a wrong number in front of the owner or the company's stakeholders. Your job:

1. Interview the person until you can fill the brief. Never invent a business fact, and never invent a metric definition: a number whose meaning the owner has not agreed is not built.
2. Build the system in their tools from the files in this folder.
3. Prove both reports against **last month**, number by number, before anything is enabled. Leave both routines **disabled** and both modes on **Paused**, with a checklist that takes the owner to Live.

Read [`../_shared/INTERVIEW.md`](../_shared/INTERVIEW.md), [`../_shared/AIRTABLE.md`](../_shared/AIRTABLE.md), and [`../_shared/ROUTINES.md`](../_shared/ROUTINES.md) before you start. They are binding.

## What you are building

| Part                           | What it does                                                                                                                                                                                                                                        | Runs as                                               |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| **1. Metric definitions**      | Every number the company reports, defined once in its own words: what counts, the source, the formula, the window, the wording rules. Both reports read the same rows, so the pulse and the deck can never disagree on what a word means.           | Airtable (Metric Definitions)                         |
| **2. Metrics snapshot**        | One read-only row (or file) per month with every product number, refreshed nightly, versioned. Built by a developer agent from [`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md). Without a codebase, the routines compute at run time from named sources.     | Code cron in the company's codebase, or run-time mode |
| **3. `RP · Metrics Pulse`**    | Weekly. One chat message: growth for the month to date and all time, and a per-person table of meetings held, demos booked, and emails sent from the CRM. Posts whatever blocks it could build and names the missing source.                        | Agent routine                                         |
| **4. `RP · Stakeholder Deck`** | Monthly, on deck day only. Builds the advisor, board, or investor update from the snapshot, the CRM, the company-state document, and the decisions log; renders charts; produces the deck file; delivers it to the shared drive and a chat message. | Agent routine with code execution                     |
| **5. Control panel**           | Report Settings (Mode, destination, meeting rule, overrides), Report Runs (every run, with the deck and the frozen payload it was built from), optional Highlights.                                                                                 | Airtable interface                                    |

```
product database ──(nightly producer, SNAPSHOT_SPEC.md)──▶ metrics snapshot  (one row per month, read only)
                                                                │
CRM Activities ─────────────────────┐                           │
Metric Definitions · Report Settings ┼──▶ RP · Metrics Pulse (weekly) ──▶ one chat message
                                     │
company-state doc · decisions log · Highlights
                                     └──▶ RP · Stakeholder Deck (deck day only)
                                            payload.base.json (snapshot, protected)
                                            + crm + narrative + render  ──▶ payload.json
                                            ──▶ generator ──▶ QA render ──▶ deck folder + chat (+ a draft the owner sends)
```

## What it needs

Fill this table during Phase 0 and keep it in the brief.

| Capability                      | Used by                     | Default                                                                    | Alternatives                                                      | If missing                                                                                |
| ------------------------------- | --------------------------- | -------------------------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Operational database            | everything                  | Airtable (MCP or API)                                                      | none in this kit                                                  | stop                                                                                      |
| Agent scheduler                 | both routines               | cloud agent routines                                                       | a desktop scheduled task, any hosted agent with cron + connectors | run by hand                                                                               |
| Metrics snapshot (read only)    | both routines               | a table in the product database, read through a read-only role             | a JSON file per month in file storage                             | run-time mode: the routines compute each metric from the sources in Metric Definitions    |
| Developer agent in the codebase | the snapshot producer       | any coding agent with repository and database access                       | a developer working from `SNAPSHOT_SPEC.md`                       | run-time mode                                                                             |
| CRM read                        | pulse team table, deck GTM  | the Sales Pipeline kit's Airtable CRM                                      | another CRM through its connector, with a field map in the brief  | the team table and the go-to-market slide are off                                         |
| Code execution                  | the deck                    | Node (`pptxgenjs`) or Python (`python-pptx`), Pillow, LibreOffice, poppler | the runtime's presentation skill                                  | the deck routine posts its summary message only; no file                                  |
| File storage, read and create   | outline, logos, deck folder | Google Drive                                                               | Dropbox, OneDrive or SharePoint, S3                               | the deck is returned with the routine's result and the message says it is not in a folder |
| Shared docs                     | outline, company-state doc  | Google Docs                                                                | Notion, a repository file                                         | the outline lives as a long-text field; narrative slides are off                          |
| Decisions log (read only)       | deck decisions slide        | an Airtable decisions table                                                | a decisions document                                              | the decisions slide is off                                                                |
| Chat                            | both routines               | Slack                                                                      | Teams, Discord                                                    | the Fallback delivery automation emails the owner                                         |
| Mail drafts (optional)          | deck to outside recipients  | Gmail drafts                                                               | Microsoft 365 drafts                                              | the owner forwards the deck by hand                                                       |
| Run-time sources (optional)     | run-time mode               | the payment processor, a spreadsheet, the analytics tool, read only        | any API with a read-only key                                      | that metric prints "not measured"                                                         |

## Ground rules

- **No number is ever typed.** Every number comes from the snapshot, a CRM count, or a source named in Metric Definitions, read during the run. Not from memory, a previous message, a previous deck, or a prompt.
- **One definition per word.** The pulse and the deck read the same Metric Definitions. A definition changes by a new version with an Effective From, never by an edit in place.
- **Missing is not zero.** A source that cannot be read drops its block or slide, and the report says which source. `null` means "could not measure" and prints that way.
- **The snapshot is prod truth.** Later steps add keys. They never overwrite a snapshot key. If a document disagrees with the snapshot on a product number, the snapshot wins and the owner is told privately.
- **Time.** One home time zone. Period labels and month-to-date filters state it explicitly. The deck-day guard computes in it.
- **Count to the end.** Every count pages to the last page, or uses a total the tool returns for the exact filter.
- **Nothing leaves the company without the owner's hand.** Default delivery: the shared drive and the owner's DM. If the brief names outside recipients, the routine creates a draft; the owner sends it. No routine sends email, and no routine changes who can see a file.
- **Untrusted input** ([`../_shared/ROUTINES.md`](../_shared/ROUTINES.md)) applies to every CRM note, document, and Airtable cell.

## Phase 0 — Discover

1. List your tools. Fill the capability table. Mark each row connected, available but not connected, or missing.
2. **CRM.** Search Airtable for a CRM base. If the Sales Pipeline kit's brief exists, read its Company section and its ID map: Activities, Team, Engagements, and their choice names are already known. Otherwise, read the CRM's schema and note which fields could play Activity Type, Direction, Classification (noise and internal), Actor, and Date.
3. **Existing reports.** Search the drive for previous stakeholder decks, monthly updates, and board packs. Note the slide order, the look, and the audience. Never copy a number from them.
4. **Narrative sources.** Search for a canonical company-state document (often dated copies in one folder), a decisions log (a table or a document), board minutes, and a brand pack or design tokens (the Content OS kit's brand pack, a design-system package).
5. **Snapshot.** If you can see the codebase or the database: look for an existing reporting table, view, or nightly export. Note where scheduled jobs live and how database roles are set up. Do not change anything.
6. **Run-time sources.** Note which of the payment processor, spreadsheets, and analytics tools are connected, read only.

## Phase 1 — Interview

Run Block 0 from [`../_shared/INTERVIEW.md`](../_shared/INTERVIEW.md), then these blocks in order. Defaults are in brackets.

### Block A — Audiences and cadence

1. Who reads the weekly pulse, and where should it land? [the owner's DM]
2. Who is the monthly deck for (advisors, a board, investors, a lender, a funder), and what do they use it for? Name the audience the way you say it. [one audience; a second audience is a second deck row and a second copy of the routine, built later]
3. When is the meeting? A rule (the last Friday of the month, the second Tuesday, the 15th) or "it moves every month". [last Friday of the month]
4. How many working days before the meeting should the deck arrive? [1]
5. Which month does the deck report: the month the meeting falls in, to date, or the last full month? [meeting in the last 10 days of a month: that month to date; otherwise the previous full month]
6. The confidentiality line for every slide footer. [`Confidential — prepared for <audience>`]

### Block B — Metrics, in your words

Work through every number either report will show. For each, record in the brief's metric table: the **name** as the person says it, the **definition** (what counts, what does not), the **source** and exactly where in it, the **formula**, what **new** and **cumulative** mean, the **unit**, whether it is in the pulse, the deck, or a deck headline (at most 6), and any **wording rule**.

Offer this starting list. The person renames, drops, or adds rows:

| Metric (rename it)         | Typical definition [default]                                                                               |
| -------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Sign-ups                   | People who created an account in the period. Excludes test and internal accounts.                          |
| <The things people create> | Listings, locations, stores, projects: whatever the product's supply is. A different metric from sign-ups. |
| Customers                  | People or organisations who paid at least once.                                                            |
| Revenue                    | Payments in the one status that means money moved, in the period, per currency.                            |
| Orders or bookings         | Created in the period, by status. Completed is its own number.                                             |
| Active users               | Your definition of active, in the period.                                                                  |
| Usage                      | Requests, sessions, or the product's own unit of use.                                                      |

Then ask:

1. People and things: are the people who sign up, the people who hold a role, and the things they create three different numbers? Name each. [yes, three names; never one label for two of them]
2. Revenue: which payment status means money actually moved? How are refunds, credits, waivers, and pilots treated? What is the reporting currency? Is "pre-revenue" or similar true and wanted when revenue is zero? [only the moved-money status; credits and waivers are not revenue; zero prints as 0]
3. What is excluded everywhere: test accounts, internal users, merged duplicates, archived rows, bots? [test and internal accounts, duplicates]
4. Is there a breakdown by role or plan? Can one person hold several? [count in each; totals are distinct people]
5. When did collection start for any metric that is new? [record a `measured since` date]
6. Words that must never describe a number. [issued is not active; signed up is not customer; listed is not booked; never "through the platform" for money that did not move]

### Block C — Where the numbers come from

1. Is there a product database and a codebase a developer agent may work in? [yes: build the snapshot from `SNAPSHOT_SPEC.md`]
2. Does a reporting table, view, or nightly export already exist? [if yes: read it, map its keys to the metric table, add only what is missing]
3. No codebase or no database: which source holds each metric (payment processor, spreadsheet and tab, analytics property and report)? Each becomes a run-time source, read only. [run-time mode]
4. Which connector and role will the routines use to read the snapshot? [a read-only role; never an admin key; never a role every product user holds]
5. Who applies database changes to production? [the owner or the person they name; the developer agent prepares, a person applies]
6. Where are the brand's colours, fonts, and logo files? Which logo is for dark backgrounds, and is it truly white? [read from the brand pack or design tokens Phase 0 found]

### Block D — Team activity (pulse)

1. Who appears in the team table, and with what display name? [every Team member who sells or meets counterparties]
2. Is the Sales Pipeline kit's CRM installed? [yes: use its Activities contract] If not, which CRM, and which fields mean activity type, direction, noise or internal, who did it, and the date? Record the map.
3. The three columns. Accept or change each:

   | Column        | Default rule                                                                                                           |
   | ------------- | ---------------------------------------------------------------------------------------------------------------------- |
   | Meetings held | Activity Type Meeting, dated on or before today. Count calls as meetings? [no]                                         |
   | Demos booked  | Activity Type Meeting Scheduled whose Notes contain "demo". Or: a Demo activity type, if the team logs demos that way. |
   | Emails sent   | Activity Type Email Sent or Email Reply, Direction Outbound.                                                           |

4. Which classifications are left out of every count? [Noise, Internal, System]
5. Any other column? [none]

### Block E — The deck

1. Format: pptx, Google Slides, or PDF? Who edits it after delivery? [pptx; the owner edits a copy if needed]
2. Walk the default slide list in [`DECK_OUTLINE_TEMPLATE.md`](DECK_OUTLINE_TEMPLATE.md), section 4. Turn each slide on or off, reorder, rename. [all on except Risks]
3. The look: colours, fonts, logo files. [from Block C.6]
4. File name and folder. [`<Company> <Audience> Update - <Period label>.pptx` in an `<Audience> Updates` folder with a `Dry runs` subfolder]
5. The appendix slide of sources and definitions. [on]
6. A candid "Risks and open items" slide? From which source? [off]

### Block F — Narrative sources

1. Is there one canonical document that says where the company stands (team, financing, programmes, deadlines)? Where is it, and how are versions kept (dated copies, one living document)? [none]
2. Which section feeds which slide (leadership, capital, corporate, governance, notable)?
3. Which sections are internal and must never reach a deck (unreconciled items, sensitive negotiations, anything said in confidence)? [record them by name]
4. Is there a decisions log? Which status means decided? [none; if one exists, "Locked" or its equivalent]
5. Do you want a Highlights table for things no system records? [on if there is no company-state document]
6. How long is a new team member "new" on the leadership slide? [60 days]
7. Authority order when sources disagree. [signed agreement, then the company-state document, then current system state, then memory; for product, transaction, usage, and revenue numbers, the snapshot outranks everything]
8. Known distortions in the CRM pipeline to footnote (a bulk re-tag, a stage used two ways). [none]

### Block G — Delivery and approval

1. The shared folder for decks, and who can see it. [a new folder the owner creates and shares]
2. Where the "deck ready" message goes. [the owner's DM]
3. Does anyone outside the company receive the deck by email? Name each recipient and address. [nobody: the owner forwards it] If yes: the routine creates one draft; the owner attaches or shares the deck and sends it. Sending is the approval. No routine sends.
4. Who may see the reporting base (it holds revenue and per-person activity)? [the owner and the people named here]
5. If chat is unavailable: email the owner the message through an Airtable automation? [yes]

### Block H — Schedule

| Item                    | Default                                                                           |
| ----------------------- | --------------------------------------------------------------------------------- |
| Snapshot producer       | nightly, at least 90 minutes before the earliest routine run                      |
| `RP · Metrics Pulse`    | Mondays 09:05 in the home time zone                                               |
| `RP · Stakeholder Deck` | every working day at 08:10 in the home time zone; the guard acts on deck day only |
| Snapshot max age        | 36 hours                                                                          |
| Last-month line         | on: in the first 7 days of a month, the pulse adds last month's numbers           |

## Phase 2 — Brief

Write `BRIEF.md` (or the doc or record the person chose) in the layout from [`../_shared/INTERVIEW.md`](../_shared/INTERVIEW.md). Kit sections:

- **Audiences and cadence**: pulse destination; deck audience, meeting rule, lead days, report period, confidentiality line.
- **Metric table**: one row per metric with every Block B field. This becomes Metric Definitions.
- **Sources**: snapshot mode (table, file, or run-time), read path, role, who applies production changes; run-time sources; brand assets.
- **Team activity**: tracked people with their CRM Team record ids, the CRM map if it is not the Sales Pipeline kit, the column rules, the excluded classifications.
- **Deck**: format, slide list, look, file name, folder, appendix, risks slide.
- **Narrative**: company-state document and how to find its newest version, the section map, the internal sections, the decisions log, Highlights on or off, the new-person window, the authority order, CRM caveats.
- **Delivery**: folder, deck message target, external recipients (default none), who may see the base, chat fallback.
- **Schedule**: the Block H table.

Get an explicit "go" on the brief before Phase 3.

### Placeholder map

| Source                 | Placeholders                                                                                                                                                                                                                                                                                      |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Block 0                | `COMPANY`, `OWNER_NAME`, `OWNER_EMAIL`, `OWNER_CHAT_TARGET`, `TIME_ZONE`, `WORKING_DAYS`, `WORKING_WEEKDAYS`, `HOME_HOLIDAYS`, `HOLIDAY_DATES`, `ID_PREFIX`, `SENDER_NAME`                                                                                                                        |
| Block A                | `PULSE_TARGET`, `DECK_AUDIENCE`, `DECK_TITLE`, `MEETING_RULE_TEXT`, `LEAD_WORKING_DAYS`, `REPORT_PERIOD_TEXT`, `CONFIDENTIALITY_LINE`                                                                                                                                                             |
| Block B                | the metric table; `ZERO_REVENUE_LABEL`, `COMPILED_METRIC_VERSIONS`                                                                                                                                                                                                                                |
| Block C                | `SNAPSHOT_TABLE` or `SNAPSHOT_FILE` (flags and values), `SNAPSHOT_READ_QUERY`, `SNAPSHOT_FOLDER`, `SNAPSHOT_CONNECTOR`, `SNAPSHOT_SCHEMA_VERSIONS`, `RUNTIME_METRICS`, `RUNTIME_METRIC_FORMULAS`, colour, font, and logo placeholders in the outline, `LOGO_RECOLOUR`                             |
| Block D                | `CRM_BLOCK`, `TRACKED_PEOPLE`, `EXCLUDED_CLASSIFICATIONS`, `DEMO_RULE`, `DEMO_RULE_SHORT`, `DEMO_RULE_READS_NOTES`, `COUNT_CALLS_AS_MEETINGS`, `EXTRA_TEAM_COLUMNS`                                                                                                                               |
| Block E                | `DECK_FORMAT`, `DECK_FILE_NAME`, `DECK_FOLDER`, `GENERATOR_RUNTIME`, `GENERATOR_FILE`, `GENERATOR_COMMAND`, `VALIDATE_COMMAND`, `RISKS_SLIDE`, `RISKS_OMIT`, `APPENDIX_OMIT`, slide titles                                                                                                        |
| Block F                | `COMPANY_STATE_DOC`, `COMPANY_STATE_DOC_FIND`, `COMPANY_STATE_INTERNAL_SECTIONS`, `SECTION_…`, `DECISIONS_LOG`, `DECIDED_STATUSES`, `HIGHLIGHTS`, `NEW_PERSON_WINDOW_DAYS`, `NON_SALES_ENGAGEMENT_TYPES`, `PIPELINE_STAGES`, `CRM_METRIC_FORMULAS`, `NAMED_CUSTOMERS_RULE`, `NEVER_IN_DECK_EXTRA` |
| Block G                | `MAIL_DRAFTS`, `EXTERNAL_RECIPIENTS`                                                                                                                                                                                                                                                              |
| Block H                | `PULSE_SCHEDULE`, `DECK_RUN_TIME`, `SNAPSHOT_MAX_AGE_HOURS`, `PREVIOUS_MONTH_LINE`, `PREVIOUS_MONTH_METRICS`, `PRODUCER_TIME`                                                                                                                                                                     |
| ID map (after Phase 4) | `REPORTING_BASE_ID`, `CRM_BASE_ID`, `PULSE_SETTINGS_RECORD_ID`, `DECK_SETTINGS_RECORD_ID`, `TBL_…`, `FLD_…`, `IDS_PULSE`, `IDS_DECK`, `GENERATOR_CODE`                                                                                                                                            |

A name used in an `{{#if …}}` block is a flag: on when the brief turned the feature on. Remove the whole block when it is off.

## Phase 3 — The snapshot

**With a codebase (recommended).** Hand a developer agent [`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md), the brief, and the Metric Definitions rows whose Source is `Snapshot`. It builds the producer, the notes table, the manual run, and the tests, and hands back the read query, the role, the schema version, and the key map.

- A person applies the production database change. The developer agent prepares it; it never applies it to production itself unless the brief says that is allowed.
- Backfill the previous full month with the manual run. The dry run in Phase 6 needs it.
- From the routine's own connector, run the read query. One row must come back. Zero rows means the connector reads as the wrong role: fix the role, never widen the table's access.
- Fill each Snapshot row's Period Path and Cumulative Path in Metric Definitions from the key map.

**Without one (run-time mode).** Set Source on each metric to its real source and write its Formula exactly: the connector, the call, the filter, the window in `{{TIME_ZONE}}`, pagination, and the aggregation. Test each formula by hand for last month and show the person the number and where it came from. Every run-time number is printed with its source and the time it was read.

Both modes can mix: snapshot for product numbers, run-time for revenue from the payment processor, for example.

## Phase 4 — Build

1. **The base.** Follow [`SCHEMA.md`](SCHEMA.md) and [`../_shared/AIRTABLE.md`](../_shared/AIRTABLE.md). Seed both Report Settings rows (Mode Paused), one Metric Definitions row per metric (Version 1, Effective From = the first period you will report), and Highlights if on. Write the ID map into the brief.
2. **The outline.** Fill [`DECK_OUTLINE_TEMPLATE.md`](DECK_OUTLINE_TEMPLATE.md) with the owner. Store it where the brief says. Put its URL on the Deck settings row. Version 1.0.
3. **The deck folder.** Ask the owner to create it and its `Dry runs` subfolder, with the sharing they want. Record both on the Deck row and in the ID map. Put the logos where the outline says.
4. **The generator.** Write it to the contract in [`routines/stakeholder-deck.md`](routines/stakeholder-deck.md) (Generator). Run it on three payloads: last month's real one, a sparse one, and one with long names. Render each with the routine's QA step. Show the owner the slides. Iterate on the look until they approve it. Then freeze it: the routine embeds it verbatim.

Hand the person [`MANUAL.md`](MANUAL.md) with live links for the UI-only steps.

## Phase 5 — Routines

Assemble each routine prompt as one self-contained text. A routine must not depend on reading this repository at run time.

| Routine                 | Template                                                       | Schedule (Block H)              | Connectors (least privilege)                                                                                                                                                    |
| ----------------------- | -------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RP · Metrics Pulse`    | [`routines/metrics-pulse.md`](routines/metrics-pulse.md)       | weekly                          | Airtable; the snapshot (read only); chat; run-time sources (read only). **Deny** database writes, CRM writes, mail.                                                             |
| `RP · Stakeholder Deck` | [`routines/stakeholder-deck.md`](routines/stakeholder-deck.md) | every working day, guard inside | Airtable; the snapshot (read only); file storage (read, create; **deny** delete, move, overwrite, sharing); chat; mail drafts only if Block G names recipients (**deny** send). |

For each one:

1. Fill every `{{…}}` from the brief and the ID map. Compile each CRM and run-time metric's formula from Metric Definitions, with field and choice ids, and list `<Key> v<Version>` in the compiled-versions constant. Expand `{{#each}}` and `{{#if}}` blocks. Search for `{{` and fix what remains.
2. Create it **disabled**, with the connectors and tool policies above. Give the deck routine a code-execution environment with the generator's runtime, LibreOffice, and poppler; add the logo host and the file-storage download domain to its network allowlist if the sandbox blocks them.
3. Record each routine's id in the brief.

## Phase 6 — Dry run against last month

Run both reports for the last full month, by hand, and check every number with the owner.

1. Set **Test Period** = last month (`YYYY-MM`) on the Pulse row. Run `RP · Metrics Pulse` by hand. It posts a `[TEST]` message to the owner's DM for the whole month.
2. Set **Test Period** = last month on the Deck row. Run `RP · Stakeholder Deck` by hand. It builds the deck into `Dry runs` and DMs the owner.
3. **Reconcile, number by number.** For each number in the message and the deck, open its source (the snapshot row, a CRM view filtered to the same window in `{{TIME_ZONE}}`, the processor's dashboard, the sheet) and compare. Write the result in the brief: `<Key>: matches` or `<Key>: differs, because …`.
4. A difference is fixed in the **definition or the formula**, never by editing the number: correct Metric Definitions (a new Version), refill the routine, run the test period again.
5. The owner reads the whole deck: the omissions, the wording, the narrative, the appendix. Fix the outline (a new version) or the generator, not the payload.
6. When every number matches and the owner approves the deck, enable both routines with Mode **Dry Run**. After one real pulse and one real deck day in Dry Run, the owner moves each Mode to **Live**.

## Start small

The smallest version that pays for itself: **Metric Definitions and the pulse**, in run-time mode or on a snapshot that already exists. It needs no deck tooling and is useful the first Monday. Add the snapshot producer next, then the deck.

## Definition of done

- The brief is approved and stored where the person chose. Every `UNKNOWN` lists what is switched off because of it.
- Every metric in the brief has an Active Metric Definitions row with a definition the owner agreed.
- The snapshot (or every run-time formula) returns last month's numbers, and Phase 6 recorded a match or an explained difference for each.
- The outline v1.0 is approved, ends with `END OF DECK OUTLINE`, and its URL is on the Deck row.
- The generator is frozen in the deck routine and produced an approved test deck for last month.
- Every [`MANUAL.md`](MANUAL.md) item is done or explicitly deferred by the person.
- Both routines exist, filled (no `{{`), least privilege, disabled or in Dry Run as the owner chose.
- [`OPERATORS.md`](OPERATORS.md) has been filled from the brief and handed to the person.

## Hand back

End with a short message to the person:

1. What was built: links to the base, the interface, the outline, the deck folder, the routines, and the test deck.
2. Which mode each report is in, and the next deck day the guard computed.
3. The Phase 6 reconciliation: how many numbers matched, and every explained difference.
4. The [`MANUAL.md`](MANUAL.md) items still open.
5. The open items from the brief, and what is switched off until each is answered.
