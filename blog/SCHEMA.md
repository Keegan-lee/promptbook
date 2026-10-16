# Blog writing — schema

You are building the backlog the writing engine runs on. Follow `_shared/AIRTABLE.md` for build order, identity, select options, and how agents write. This file is the field contract. Substitute the brief's vocabulary wherever a select says "from the brief".

## Choose a model

| Model                                 | When                                                                                                    | Dedupe key                                                       |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Dedicated tables** (recommended)    | Anyone starting fresh, or anyone whose task table is shared with engineering or ops work.               | A structured `Primary Keyword` field, normalised by a formula.   |
| **Existing task table** (alternative) | A team that already runs every ticket in one table and wants blog work beside it. See the last section. | A `- **Primary keyword:** …` line parsed out of the description. |

A structured field beats a parsed line. A person who edits a description can break the line without noticing, and every later run then misses a duplicate. Whichever model you pick, build the **Keyword Ledger**, the **Blog Settings** row, and the **Run Log**.

## Build order

1. Blog Settings (scalar fields only).
2. Keyword Ledger (without the Ticket link).
3. Blog Backlog (scalar fields and selects).
4. Run Log (without the Tickets link).
5. Links, created from one side: Blog Backlog → Keyword Ledger (`Keywords`), Run Log → Blog Backlog (`Tickets`).
6. Formulas: Ticket ID, Primary Keyword Key, Ledger Key, Is Stale, Next Manual Step.
7. Seed records: the one settings row, and the brief's excluded terms in the ledger.
8. Write the ID map into the brief, including the settings row's record id.

Create every select option below **before** any routine or automation writes to the table. Routines never add options.

## Vocabularies

| Select                       | Options                                                                                                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Kind                         | Geo, Depth, Refresh, News, Correction, plus each kind the brief defines (for example Glossary, Comparison, Customer Story)                                 |
| Status                       | Proposed, Ready, Writing, Draft Ready, Changes Requested, Live, Blocked, Parked, Rejected                                                                  |
| Priority                     | High, Medium, Low                                                                                                                                          |
| Refresh Scope                | Confirm, Update, Expand                                                                                                                                    |
| Source                       | Gap Scan, Refresh Review, Person, Content OS                                                                                                               |
| Mode                         | Paused, Dry Run, Live                                                                                                                                      |
| Ledger Status                | Candidate, Targeted, Rejected, Zero-volume verified                                                                                                        |
| Rejection Reason             | Low volume, High difficulty, SERP brands, SERP mismatch, Duplicate, Cannibalises live post, In progress, Excluded topic, Supply failed, Double-read failed |
| Tool                         | the brief's keyword tool, plus Search Console and Other                                                                                                    |
| Routine                      | BL · Content Gap Scan, BL · Post Writer, BL · Refresh Review, Build                                                                                        |
| Outcome                      | OK, Quiet, Shortfall, Stopped, Error                                                                                                                       |
| Error Status                 | New, Investigating, Resolved, Ignored                                                                                                                      |
| Cluster, Archetype, Audience | from the brief, verbatim                                                                                                                                   |
| CTA                          | every id in the Site brief's CTA catalog, plus `none`                                                                                                      |

**Mode** has three rungs, not the four in `_shared/ROUTINES.md`. Nothing in this kit publishes, so `Drafts` and `Live` are the same rung: "Live" means the routine makes its real writes (tickets, CMS drafts). Publishing is always a person.

## Blog Backlog

One row per post to write, refresh, or correct. Primary field **Title**.

| Field               | Type                  | Notes                                                                                                                                                       |
| ------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Title               | single line           | Working title. A `[TEST] ` prefix routes the ticket to the test dataset.                                                                                    |
| Ticket Number       | autonumber            |                                                                                                                                                             |
| Ticket ID           | formula               | `"{{ID_PREFIX}}-B" & RIGHT("0000" & {Ticket Number}, 4)`. People search this.                                                                               |
| Kind                | single select         | Vocabulary above.                                                                                                                                           |
| Status              | single select         | Vocabulary above. Allowed moves below.                                                                                                                      |
| Priority            | single select         |                                                                                                                                                             |
| Primary Keyword     | single line           | Lowercase, trimmed, no trailing punctuation. **The dedupe key.** Routines write it normalised. People may type it.                                          |
| Primary Keyword Key | formula               | `LOWER(TRIM({Primary Keyword}))`. Routines match on this, never on the raw field.                                                                           |
| Secondary Keywords  | long text             | One per line: `<keyword> · vol <n> · diff <n> (<tool>, loc <id>, <YYYY-MM-DD>)`.                                                                            |
| Keywords            | link → Keyword Ledger | Every ledger row this ticket targets.                                                                                                                       |
| Cluster             | single select         | From the brief.                                                                                                                                             |
| Archetype           | single select         | From the brief. Decides the recipe the writer follows.                                                                                                      |
| Audience            | single select         | From the brief.                                                                                                                                             |
| Category            | single line           | The CMS category slug. Must exist in the CMS.                                                                                                               |
| CTA                 | single select         | A catalog id, or `none`.                                                                                                                                    |
| Target Place        | single line           | Geo tickets only. The place name as the brief writes it.                                                                                                    |
| Location Id         | number                | The keyword tool's location id the numbers were pulled at.                                                                                                  |
| Target Slug         | single line           | New posts only. Proposed by the scan, checked by the writer.                                                                                                |
| Refresh Scope       | single select         | Refresh tickets only. Confirm (nothing found to change; a person re-reads and re-dates), Update (facts or sources changed), Expand (add coverage).          |
| Existing Doc Id     | single line           | Refresh and Correction: the published CMS id. Never a draft id.                                                                                             |
| Existing URL        | url                   | Refresh and Correction: the live URL.                                                                                                                       |
| Brief               | long text             | The ticket body from `TICKET_TEMPLATE.md`.                                                                                                                  |
| Evidence            | long text             | Supply evidence (the query, both reads, the date), SERP notes, refresh findings. Internal only. Counts here never go into copy.                             |
| Draft Markdown      | long text             | Dry Run output from the writer: the post as it would be written, plus the guard results.                                                                    |
| CMS Document Id     | single line           | The document id without any draft prefix.                                                                                                                   |
| CMS Dataset         | single line           | Production or the test dataset.                                                                                                                             |
| Draft URL           | url                   | The CMS editor link for the draft.                                                                                                                          |
| Preview Path        | url                   | The post's public URL without any preview secret. The reviewer adds the secret, or uses the CMS's own preview.                                              |
| Live URL            | url                   | Set when the post is published and live.                                                                                                                    |
| Draft Written At    | date-time             | Home time zone. When the writer last wrote the draft.                                                                                                       |
| Live At             | date-time             | Stamped by automation when Status becomes Live.                                                                                                             |
| Revision Count      | number, integer       | Writer revisions after Changes Requested. Capped at 2.                                                                                                      |
| Reviewer            | single collaborator   | Who reviews the draft. Defaults from the brief.                                                                                                             |
| Approved By         | single collaborator   | Stamped by automation when a signed-in person sets Status to Ready. An API write never sets it.                                                             |
| Approved At         | date-time             | Same stamp.                                                                                                                                                 |
| Review Notes        | long text             | People write here. The writer reads it on Changes Requested and Blocked.                                                                                    |
| Agent Notes         | long text             | Append-only, newest line on top: `YYYY-MM-DD · <routine>: <what> (<evidence>)`.                                                                             |
| Changed By          | single line           | The routine or person behind the last change. Many teams share one login, so Airtable's own history cannot say.                                             |
| Changed At          | date-time             | With Changed By.                                                                                                                                            |
| Source              | single select         | Who created the row.                                                                                                                                        |
| Run Ref             | single line           | `<routine> <YYYY-MM-DD>` of the run that created it.                                                                                                        |
| Content Ref         | single line           | Optional. The Content OS id (for example `{{ID_PREFIX}}-C0042`) when the kit is installed in another base. Use a link field instead if it is the same base. |
| Next Manual Step    | formula               | Below.                                                                                                                                                      |
| Created             | created time          |                                                                                                                                                             |
| Last Modified       | last modified time    |                                                                                                                                                             |

### Status moves

Routines move a status only along these arrows. Nothing else moves backward.

| From              | To                      | Who                                                                                       |
| ----------------- | ----------------------- | ----------------------------------------------------------------------------------------- |
| (new)             | Proposed                | Gap Scan, Refresh Review, a person                                                        |
| Proposed          | Ready, Rejected, Parked | a person (Ready is stamped)                                                               |
| Proposed          | Writing                 | the writer, only when Blog Settings → Auto-Ready Topics is on                             |
| Ready             | Writing                 | the writer (claims the row)                                                               |
| Writing           | Draft Ready, Blocked    | the writer                                                                                |
| Writing           | Ready                   | the writer, releasing a claim older than 24 hours from a failed run                       |
| Draft Ready       | Changes Requested       | a person, with Review Notes                                                               |
| Draft Ready       | Live                    | the writer's reconcile step, after reading the CMS; or a person                           |
| Draft Ready       | Blocked                 | the writer's reconcile step, when the draft is gone and nothing is published              |
| Changes Requested | Writing                 | the writer, while Revision Count is below 2                                               |
| Changes Requested | Blocked                 | the writer, when Revision Count has reached 2, or the reconcile step finds the draft gone |
| Changes Requested | Live                    | the writer's reconcile step, when a person published the draft anyway                     |
| Blocked           | Ready                   | a person, after fixing the cause                                                          |
| Parked            | Proposed                | a person                                                                                  |

### Next Manual Step

```
IF(OR({Status} = "Rejected", {Status} = "Parked", {Status} = "Live"), "",
IF({Status} = "Blocked", "1 · Unblock: read Agent Notes, fix, set Ready",
IF({Status} = "Draft Ready", "2 · Review the draft, publish, switch live on",
IF(AND({Status} = "Ready", LEN({Approved By} & '') = 0, LEFT({Title}, 6) != "[TEST]"), "3 · Set Ready yourself in the UI so it is stamped",
IF({Status} = "Proposed", "4 · Approve or reject the topic",
"")))))
```

Blank means nothing is waiting on a person.

## Keyword Ledger

One row per keyword per location per language. It keeps paid numbers so later runs reuse them, and keeps verified dead ends out of every future run. Primary field **Keyword**.

| Field            | Type                | Notes                                                                                                                |
| ---------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------- | --------------------- | --------------------------------------------- |
| Keyword          | single line         | Lowercase, trimmed.                                                                                                  |
| Location Id      | number              | From the tool. Never guessed.                                                                                        |
| Location         | single line         | The place name the id resolved to.                                                                                   |
| Language         | single line         | For example `en`.                                                                                                    |
| Ledger Key       | formula             | `LOWER(TRIM({Keyword})) & "                                                                                          | " & {Location Id} & " | " & LOWER({Language})`. For views and search. |
| Volume           | number              | Blank when the tool gave a range or nothing.                                                                         |
| Volume Range     | single line         | For tools that return ranges, for example `10–100`.                                                                  |
| Difficulty       | number              | Blank when the tool has none.                                                                                        |
| CPC              | number, 2 decimals  |                                                                                                                      |
| Tool             | single select       |                                                                                                                      |
| Pulled At        | date                | The day the numbers were pulled.                                                                                     |
| Is Stale         | formula             | `IF(IS_BEFORE({Pulled At}, DATEADD(TODAY(), -{{LEDGER_TTL_DAYS \| 90}}, 'days')), 1, 0)`. Hard-code the brief's TTL. |
| Ledger Status    | single select       | Candidate, Targeted, Rejected, Zero-volume verified.                                                                 |
| Rejection Reason | single select       | Required when Ledger Status is Rejected.                                                                             |
| Cluster          | single select       | From the brief.                                                                                                      |
| SERP Notes       | long text           | Who ranks, content type that wins, small sites yes or no, with tool and date.                                        |
| Ticket           | link → Blog Backlog | Inverse of Backlog → Keywords.                                                                                       |
| Post URL         | url                 | The live post that targets it, when one does.                                                                        |
| Run Ref          | single line         |                                                                                                                      |
| Agent Notes      | long text           | Append-only.                                                                                                         |

Rules:

- **Upsert** on `Keyword`, `Location Id`, and `Language` together (`performUpsert`, three merge fields). Lowercase and trim the keyword before writing.
- **Zero-volume verified** rows are never queried again and never targeted. Only a person changes that status. Seed the brief's verified dead ends here at build, each with its tool and date.
- **Targeted** rows are not overwritten by a later run except to refresh Volume, Difficulty, CPC, and Pulled At after the TTL.
- **Rejected** rows are reused within the TTL. After the TTL a run may re-check them; a keyword's fortunes change.

## Blog Settings

Exactly one row, named `Blog Settings`. The brief is the source of truth; this row is its live copy for the routines. Change the brief first, then the row (`_shared/INTERVIEW.md`, rule 10).

| Field                   | Type          | Default from the brief                                           |
| ----------------------- | ------------- | ---------------------------------------------------------------- |
| Name                    | single line   | `Blog Settings`                                                  |
| Gap Scan Mode           | single select | Paused                                                           |
| Writer Mode             | single select | Paused                                                           |
| Refresh Mode            | single select | Paused                                                           |
| Kill Reason             | long text     | Written by a guard that lowers a mode, or by the owner.          |
| Weekly Target           | number        | `{{WEEKLY_TARGET \| 7}}`                                         |
| Weekly Minimum          | number        | `{{WEEKLY_MINIMUM \| 4}}`                                        |
| Mix Geo                 | number        | `{{MIX_GEO \| 3}}`                                               |
| Mix Depth               | number        | `{{MIX_DEPTH \| 2}}`                                             |
| Mix Refresh             | number        | `{{MIX_REFRESH \| 1}}`                                           |
| Mix News                | number        | `{{MIX_NEWS \| 1}}`                                              |
| Public Facts Geo Cap    | number        | `{{TIER3_CAP \| 1}}` per run                                     |
| Min Volume              | number        | `{{MIN_VOLUME \| 30}}`                                           |
| Max Difficulty          | number        | `{{MAX_DIFFICULTY \| 35}}`                                       |
| SERP Brand Authority    | number        | `{{SERP_BRAND_AUTHORITY \| 60}}`                                 |
| Domain Authority        | number        | Read with the tool at build.                                     |
| Domain Authority Source | single line   | `<tool>, <YYYY-MM-DD>`                                           |
| Market Location         | single line   | `{{MARKET_NAME}}`                                                |
| Market Location Id      | number        | `{{MARKET_LOC_ID}}`, resolved with the tool.                     |
| Query Language          | single line   | `{{QUERY_LANGUAGE \| en}}`                                       |
| Scan Call Budget        | number        | `{{SCAN_CALL_BUDGET \| 40}}`                                     |
| Refresh Call Budget     | number        | `{{REFRESH_CALL_BUDGET \| 20}}`                                  |
| Quota Floor             | number        | `{{QUOTA_FLOOR \| 10}}`                                          |
| Ledger TTL Days         | number        | `{{LEDGER_TTL_DAYS \| 90}}`                                      |
| Reject Cooldown Days    | number        | `{{REJECT_COOLDOWN_DAYS \| 90}}`                                 |
| Writer Batch            | number        | `{{WRITER_BATCH \| 2}}`                                          |
| Writer Min Gap Hours    | number        | `{{MIN_RUN_GAP_HOURS \| 20}}`. 0 turns the check off (Phase 6).  |
| Refresh Batch           | number        | `{{REFRESH_BATCH \| 10}}`                                        |
| Auto-Ready Topics       | checkbox      | off                                                              |
| Brief URL               | url           |                                                                  |
| Brief Version           | single line   |                                                                  |
| Brand Pack URL          | url           |                                                                  |
| Last Gap Scan At        | date-time     |                                                                  |
| Last Gap Scan Summary   | long text     |                                                                  |
| Last Writer Run At      | date-time     |                                                                  |
| Last Writer Summary     | long text     |                                                                  |
| Last Refresh Review At  | date-time     |                                                                  |
| Last Refresh Summary    | long text     |                                                                  |
| Changed By              | single line   | Who last changed a setting: a person's name or a routine's name. |
| Changed At              | date-time     |                                                                  |
| Agent Notes             | long text     | Append-only.                                                     |

Only the owner raises a mode. A guard may lower one, writing the Kill Reason.

## Run Log

One row per routine run. It is the history behind the settings row's "last run" fields, the record of keyword-tool spend, and the error queue. Primary field **Run**.

| Field           | Type                | Notes                                                                            |
| --------------- | ------------------- | -------------------------------------------------------------------------------- |
| Run             | single line         | `<routine> · <YYYY-MM-DD HH:mm>` in the home time zone.                          |
| Routine         | single select       |                                                                                  |
| Started At      | date-time           |                                                                                  |
| Mode            | single select       | The mode the run used.                                                           |
| Outcome         | single select       | OK, Quiet (nothing to do), Shortfall, Stopped (gate, quota, missing doc), Error. |
| Tickets Created | number              |                                                                                  |
| Drafts Created  | number              |                                                                                  |
| Blocked         | number              |                                                                                  |
| Keyword Calls   | number              | Paid calls made this run.                                                        |
| Quota Left      | single line         | `<used>/<limit> (<tool>, <HH:mm>)`, or `not exposed`.                            |
| Summary         | long text           | The routine's run summary. Counts and ticket ids, no personal data.              |
| Error Message   | long text           | One line. No tokens, headers, or page text.                                      |
| Error Status    | single select       | New on an Error row; people move it on.                                          |
| Tickets         | link → Blog Backlog | Rows this run created or changed.                                                |

## Views

The API cannot create views. They are in `MANUAL.md`. Names and filters:

| Table          | View                    | Filter and sort                                                                                                       |
| -------------- | ----------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Blog Backlog   | All                     | none. **Do not filter, do not delete** (automations watch it).                                                        |
| Blog Backlog   | Triage                  | Status = Proposed. Priority, then Created.                                                                            |
| Blog Backlog   | Ready Queue             | Status = Ready. Priority, then Approved At.                                                                           |
| Blog Backlog   | Drafts to Review        | Status = Draft Ready. Draft Written At, oldest first.                                                                 |
| Blog Backlog   | Blocked                 | Status = Blocked.                                                                                                     |
| Blog Backlog   | Refresh and Corrections | Kind is Refresh or Correction, Status is not Live, Rejected, or Parked.                                               |
| Blog Backlog   | Live Posts              | Status = Live. Fields: Ticket ID, Title, Live URL, Primary Keyword, Audience, Cluster, Live At. Other kits read this. |
| Blog Backlog   | Pipeline                | Kanban by Status.                                                                                                     |
| Keyword Ledger | Candidates              | Ledger Status = Candidate. Volume, highest first.                                                                     |
| Keyword Ledger | Targeted                | Ledger Status = Targeted.                                                                                             |
| Keyword Ledger | Rejected                | Ledger Status = Rejected. Grouped by Rejection Reason.                                                                |
| Keyword Ledger | Dead Ends               | Ledger Status = Zero-volume verified.                                                                                 |
| Keyword Ledger | Stale                   | Is Stale = 1.                                                                                                         |
| Run Log        | Errors                  | Outcome = Error and Error Status is New or Investigating.                                                             |
| Run Log        | Recent                  | Started At, newest first.                                                                                             |

## Interface

**Blog Desk.** Drafted by the agent where the tool allows; finished and shared by a person (`MANUAL.md`).

| Page     | Shows                                                                   | People do                                                   |
| -------- | ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| Triage   | Triage view as a record review: Brief, keyword data, Evidence, Priority | Set Ready, Rejected, or Parked; adjust Priority             |
| Review   | Drafts to Review: Draft URL, Preview Path, Agent Notes, Review Notes    | Open the draft, publish, switch live on; or request changes |
| Pipeline | Kanban by Status, filtered to not Live, Rejected, Parked                | See the week                                                |
| Blocked  | Blocked view with Agent Notes                                           | Fix, then set Ready                                         |
| Keywords | Ledger views                                                            | Mark a dead end, revive a rejected keyword                  |
| Settings | The Blog Settings record                                                | Change modes and thresholds (owner only)                    |
| Runs     | Run Log, Recent and Errors                                              | Triage errors                                               |

## Automations

Created as drafts. A person turns each on after a test record proves it (`MANUAL.md`).

| Automation           | Trigger                                                                       | Action                                                                                                                                            |
| -------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Topic approval stamp | Status changes to Ready, made by a signed-in person in the UI                 | Approved By = that person, Approved At = now. Build it the way `_shared/AIRTABLE.md` ("Approval stamps") describes, so an API write never stamps. |
| Clear stale stamp    | Status changes to Proposed, Rejected, or Parked                               | Clear Approved By and Approved At.                                                                                                                |
| Live stamp           | Status changes to Live and Live At is empty                                   | Live At = now.                                                                                                                                    |
| Error email          | Run Log record created with Outcome = Error                                   | Email the owner: Run, Routine, Error Message. No other content.                                                                                   |
| Draft ready email    | Optional, only when there is no chat connector: Status changes to Draft Ready | Email the Reviewer: Ticket ID, Title, Draft URL.                                                                                                  |

## Content OS link (optional)

If the Content OS kit is installed, its nightly CMS inventory sync creates one Content row for every published and live post, keyed on the CMS Document ID (the published id, no draft prefix). That row is how a live post reaches social promotion. This kit never creates Content OS rows; it only links to them.

- Same base: add `Content` as a link field to its Content table and use it instead of `Content Ref`.
- Another base: keep `Content Ref` as text holding the Content id, and record that base's ids in the brief's ID map. Airtable links cannot cross bases.
- The writer's reconcile step looks the Content row up by CMS Document ID and writes its id into Content Ref. Read-only.
- The Content OS kit's own article-to-CMS routine stays off while this kit is installed. Two writers would race for the same slugs.

## Existing task table alternative

Use this when blog tickets must live in a task table the team already runs. The routines are written for the dedicated table; at build, apply this mapping to every routine prompt and record it in the brief.

| Dedicated table                      | Existing task table                                                                                                                                                                                                                                                                                                                                         |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A Blog Backlog row                   | A task titled `Blog: <descriptive title>`.                                                                                                                                                                                                                                                                                                                  |
| Kind                                 | Tags: `blog` plus **exactly one** of `blog-geo`, `blog-depth`, `blog-refresh`, `blog-news`, `blog-correction`, and one `blog-<kind>` per brief-defined kind. Create every tag option at build.                                                                                                                                                              |
| Status                               | The table's own options, mapped once in the brief: Proposed → its intake option (for example Backlog or Needs Grooming); Ready → its to-do option; Writing → its in-progress option; Draft Ready → its review option (for example QA); Live → its done option; Blocked → its blocked option. Read the live choices every run; never write Done from a scan. |
| Primary Keyword                      | The description line, exactly: `- **Primary keyword:** <lowercase trimmed keyword>`. Do not reformat it. It is the dedupe key for every later run.                                                                                                                                                                                                          |
| Other structured fields              | Lines under the description's `## POST FIELDS` section (`TICKET_TEMPLATE.md`).                                                                                                                                                                                                                                                                              |
| CMS Document Id, Draft URL, Live URL | Lines the writer appends to the table's notes field, one per line: `CMS draft: <id> · <editor link>` and `Live: <url>`. If the table has a "later decisions" notes field that overrides the description, write there.                                                                                                                                       |
| Agent Notes                          | The table's notes field, append-only, dated lines.                                                                                                                                                                                                                                                                                                          |
| Approved By stamp                    | The table's own approval convention, if any. If none, the writer takes only tickets a person moved to the to-do option, and the brief records that there is no stamp.                                                                                                                                                                                       |

Rules for this model:

- The scan pulls every task tagged `blog` and parses the `Primary keyword:` line. A ticket whose line is missing or does not parse is listed as `DEDUPE-UNVERIFIED` in the run summary, with its id. The scan never edits it.
- Recommended: add a structured `Primary Keyword` field to the existing table anyway. Adding a field is additive and safe; the routines then prefer the field and fall back to the line.
- Keep the Keyword Ledger, Blog Settings, and Run Log as separate tables in the same base. They do not belong in a task table.
- Set the existing table's other required fields from the brief (type, source, engine, effort, priority), using options read live, never typed from memory.
