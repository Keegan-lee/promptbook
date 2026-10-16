# Reporting — schema

The field contract for the reporting control panel. Build it with [`../_shared/AIRTABLE.md`](../_shared/AIRTABLE.md). Substitute the brief's vocabulary wherever a select says **(brief)**. Routines address every field by id, through the brief's ID map.

This base is small on purpose. The numbers live in the metrics snapshot ([`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md)) and the CRM. Airtable holds the switches, the definitions, and the run history.

| Table              | Rows                                   | Who writes                                                                     |
| ------------------ | -------------------------------------- | ------------------------------------------------------------------------------ |
| Report Settings    | One per report                         | The owner (switches); routines (last-run fields only)                          |
| Metric Definitions | One per metric, per definition version | People, at the kickoff and whenever a definition changes                       |
| Report Runs        | One per run that did work              | Routines                                                                       |
| Highlights         | Optional. One per item for the deck    | People. On when the brief has no company-state document, or the team wants it. |

## Where the base lives

Base name: `{{BASE_NAME | <Company> Reporting}}`.

- **Its own base** (recommended). Report Runs and Metric Definitions describe company metrics, revenue included. Share the base with the owner and the people the brief names under "Who may see the reporting base". The CRM base is usually shared more widely.
- **Inside an existing operations base**, when the brief says so. Never inside the CRM base unless everyone with CRM access may see revenue.

The routines read the CRM from its own base. Airtable links cannot cross bases, so Tracked People holds CRM Team record ids as text (below).

## Build order

1. Report Settings (scalar fields and selects).
2. Metric Definitions (scalar fields and selects), then its self-link `Superseded By`.
3. Report Runs (scalar fields and selects), then its formula id.
4. Highlights, if on.
5. Seed records: two Report Settings rows, every metric from the brief's metric table, nothing in Report Runs.
6. Write the ID map into the brief, including both settings rows' record ids.

Create every select option below **before** any routine or automation writes to the table. Routines never add options. Date-times display in `{{TIME_ZONE}}`, 24-hour. Set the zone when you create each field: the API cannot change it afterwards.

## Vocabularies

| Select                     | Options                                                                                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Report                     | `RP · Metrics Pulse`, `RP · Stakeholder Deck`                                                                                                    |
| Mode                       | Paused, Dry Run, Live                                                                                                                            |
| Report Period              | Current month to date, Previous full month                                                                                                       |
| Meeting Rule               | Last weekday of month, Nth weekday of month, Day of month, Manual only                                                                           |
| Meeting Weekday            | Mon, Tue, Wed, Thu, Fri, Sat, Sun                                                                                                                |
| Block                      | Growth, Revenue, Marketplace, Usage, Segments, Community, CRM, Team Activity, Custom (brief: drop the ones the business does not have)           |
| Unit                       | Count, Money, Percent, Duration                                                                                                                  |
| Source                     | Snapshot, CRM, Payment Processor, Spreadsheet, Analytics, Company State Doc, Other                                                               |
| Metric Status              | Active, Proposed, Retired                                                                                                                        |
| Outcome                    | OK, Partial, Already Delivered, Stopped, Error                                                                                                   |
| Sources (Used and Missing) | Snapshot, CRM, Company State Doc, Decisions Log, Highlights, Payment Processor, Spreadsheet, Analytics, Outline, Logos, File Storage, Chat, Mail |
| Error Status               | New, Investigating, Resolved, Ignored                                                                                                            |
| Highlight Category         | Hire, Board, Partnership, Customer, Product, Funding, Other (brief)                                                                              |

**Mode** has three rungs, not four. No rung of either report sends external email: the deck routine creates email drafts at most, and the owner sends each one. Sending the draft is the approval.

| Mode    | `RP · Metrics Pulse`                                                    | `RP · Stakeholder Deck`                                                                                                                |
| ------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Paused  | Does nothing. Writes one Last Run Summary line.                         | Does nothing. Writes one Last Run Summary line.                                                                                        |
| Dry Run | Builds the message and posts it to the owner's DM only, marked DRY RUN. | On deck day, builds the deck, saves it to the `Dry runs` subfolder, DMs the owner only. No drafts. Does not mark the period delivered. |
| Live    | Posts to Destination.                                                   | On deck day, saves to the deck folder, posts to Destination, creates drafts if the brief names external recipients.                    |

Only the owner raises a Mode. A guard may lower one, writing Kill Reason.

## Report Settings

One row per report. Each routine reads its own row before it does anything else. Primary field **Report**.

| Field                  | Type          | Applies to | Notes                                                                                                                                                                                                                                                                               |
| ---------------------- | ------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Report                 | single line   | both       | Exactly `RP · Metrics Pulse` or `RP · Stakeholder Deck`.                                                                                                                                                                                                                            |
| Mode                   | single select | both       | Seed: Paused.                                                                                                                                                                                                                                                                       |
| Kill Reason            | long text     | both       | Dated line from the guard that lowered Mode or stopped a run.                                                                                                                                                                                                                       |
| Destination            | single line   | both       | The chat channel or DM for this report's message. Default: the owner's DM.                                                                                                                                                                                                          |
| Test Period            | single line   | both       | `YYYY-MM`. When set, the next run reports that whole month, ignores the deck-day guard, runs even while Mode is Paused, always delivers as Dry Run, and clears this field. Used for the kickoff's dry run and for checking a fix.                                                   |
| Tracked People         | long text     | Pulse      | One line per person: `Display Name · <CRM Team record id>`. The order of lines is the order of the table.                                                                                                                                                                           |
| Report Period          | single select | Deck       | Which month the deck reports. Default from the brief.                                                                                                                                                                                                                               |
| Meeting Rule           | single select | Deck       |                                                                                                                                                                                                                                                                                     |
| Meeting Weekday        | single select | Deck       | For the two weekday rules.                                                                                                                                                                                                                                                          |
| Meeting Nth            | number        | Deck       | 1 to 4, for "Nth weekday of month". Use "Last weekday of month" instead of 5.                                                                                                                                                                                                       |
| Meeting Day Of Month   | number        | Deck       | For "Day of month". A day past the month's end means its last day.                                                                                                                                                                                                                  |
| Lead Working Days      | number        | Deck       | Working days between deck day and the meeting. `{{LEAD_WORKING_DAYS \| 1}}`.                                                                                                                                                                                                        |
| Meeting Date Override  | date          | Deck       | This month's meeting when it moves off the rule. A past date is ignored. The routine never clears it.                                                                                                                                                                               |
| Next Deck Day          | date          | Deck       | Written by the routine on every run, so a person can check the guard.                                                                                                                                                                                                               |
| Last Delivered Period  | single line   | Deck       | `YYYY-MM`. Written by a Live delivery. A second run for the same period stops unless Build Now is ticked.                                                                                                                                                                           |
| Build Now              | checkbox      | Deck       | The owner ticks it so the next run skips the deck-day guard and builds the period of the meeting nearest today, in the row's Mode: after a missed deck day, or to rebuild a delivered period. The routine unticks it. A rebuild never overwrites a delivered file; it saves `… v2`. |
| Snapshot Max Age Hours | number        | both       | A snapshot older than this is used but flagged as stale. `{{SNAPSHOT_MAX_AGE_HOURS \| 36}}`.                                                                                                                                                                                        |
| CRM Caveat             | long text     | Deck       | A footnote printed under the pipeline counts (for example a known bulk re-tag that inflates a stage). The owner maintains it. Empty means no footnote.                                                                                                                              |
| Outline URL            | url           | Deck       | The deck outline document ([`DECK_OUTLINE_TEMPLATE.md`](DECK_OUTLINE_TEMPLATE.md)).                                                                                                                                                                                                 |
| Outline Version        | single line   | Deck       | Stamped by the routine from the outline's version line.                                                                                                                                                                                                                             |
| Deck Folder URL        | url           | Deck       | Where delivered decks go.                                                                                                                                                                                                                                                           |
| External Recipients    | long text     | Deck       | One address per line. Empty by default. The routine drafts only to an address that is in this field **and** in its own constants from the brief; an address in one place only is reported, never drafted.                                                                           |
| Last Run At            | date-time     | both       |                                                                                                                                                                                                                                                                                     |
| Last Run Summary       | long text     | both       | Newest line on top, keep the last 20. Counts, links, and the next deck day. No metric values.                                                                                                                                                                                       |
| Changed By             | single line   | both       | Who last changed a setting: a person's name or a routine's name.                                                                                                                                                                                                                    |
| Changed At             | date-time     | both       |                                                                                                                                                                                                                                                                                     |
| Agent Notes            | long text     | both       | Append-only, newest line on top: `YYYY-MM-DD · <routine>: <what>`.                                                                                                                                                                                                                  |

Seed the Pulse row with Mode Paused, Destination, Tracked People, and Snapshot Max Age Hours. Seed the Deck row with Mode Paused and every Deck field the brief answered. Leave the other row's fields blank.

## Metric Definitions

One row per metric, in the company's words. Both routines read it on every run for labels, order, and what to include. It is the live copy of the brief's metric table; change the brief first, then the row. Primary field **Metric**.

| Field           | Type                      | Notes                                                                                                                                            |
| --------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Metric          | single line               | The label as it is printed, in the company's words.                                                                                              |
| Key             | single line               | Unique, lowercase, dotted: `growth.signups`, `revenue.succeeded`, `crm.outreach_sent`, `team.emails_sent`. Never reused for a different meaning. |
| Version         | number                    | Starts at 1. Raised whenever Formula or Definition changes meaning. The routines are compiled against a version (below).                         |
| Block           | single select             |                                                                                                                                                  |
| Definition      | long text                 | Plain words: what counts, what does not. Exclusions spelled out (test accounts, internal users, duplicates).                                     |
| Formula         | long text                 | Exact: the source table or endpoint, the filter, the aggregation, the window, and the time zone.                                                 |
| Period Path     | single line               | Where the period value sits in the payload, for example `growth.signups.new`. Blank when there is no period value.                               |
| Cumulative Path | single line               | Where the all-time value sits, for example `growth.signups.cumulative`. Blank when there is none.                                                |
| Unit            | single select             | Money is stored in minor units (cents) with its currency, and printed in major units.                                                            |
| Source          | single select             | Snapshot when the producer computes it. Anything else means the routine computes it at run time from that source.                                |
| Source Detail   | single line               | Exactly where: the snapshot table, the CRM table, the processor account, the sheet and tab, the analytics property and report.                   |
| Wording Rule    | single line               | Words the label must and must not use, for example `say "issued", never "active"` or `people, never "listings"`.                                 |
| In Pulse        | checkbox                  |                                                                                                                                                  |
| In Deck         | checkbox                  |                                                                                                                                                  |
| Headline        | checkbox                  | One of at most six numbers on the deck's executive snapshot and in the "deck ready" message.                                                     |
| Order           | number                    | Sort order inside its block.                                                                                                                     |
| Metric Status   | single select             | Active, Proposed (not yet used), Retired.                                                                                                        |
| Effective From  | single line               | `YYYY-MM`. The first period this version applies to.                                                                                             |
| Superseded By   | link → Metric Definitions | Set on a Retired row.                                                                                                                            |
| Defined By      | single line               | Who agreed this definition.                                                                                                                      |
| Notes           | long text                 |                                                                                                                                                  |

**Changing a definition.** Never edit a used definition in place. Create a new row with the same Key, Version + 1, and an Effective From; set the old row to Retired with Superseded By. The deck footnotes the change the first period it applies. A definition that changes later must not silently rewrite what stakeholders were already told.

**Compiled formulas.** Labels, order, and the In Pulse, In Deck, and Headline boxes are read live on every run. Formulas for CRM and run-time metrics need field and choice ids, so the build compiles them into the routine prompts with each metric's Version. On every run a routine compares the Active rows' Versions with the versions it was compiled with. A mismatch skips that metric and reports `definition changed since build: <Key>; refill the routine`. It never guesses at a new formula.

### Seed metrics

Seed one Active row per metric in the brief. Typical rows, to be renamed in the company's words or dropped:

| Key                    | Block         | Typical definition                                                                                               |
| ---------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------- |
| `growth.signups`       | Growth        | People who created an account. Excludes test and internal accounts.                                              |
| `growth.<supply_unit>` | Growth        | The things people create or list (locations, stores, listings, projects). Never labelled as sign-ups.            |
| `revenue.succeeded`    | Revenue       | Payments in the one status that means money moved, in the period. Minor units, per currency.                     |
| `marketplace.orders`   | Marketplace   | Orders or bookings created in the period, with a by-status breakdown. Completed is its own number.               |
| `usage.active_users`   | Usage         | The brief's definition of active, in the period.                                                                 |
| `crm.outreach_sent`    | CRM           | Activities, Activity Type Email Sent, Direction Outbound, in the period.                                         |
| `crm.responses`        | CRM           | Activities, Activity Type Inbound Email or Email Reply, Direction Inbound, in the period.                        |
| `crm.meetings_held`    | CRM           | Activities, Activity Type Meeting (or Call, if the brief counts calls), dated in the period and not after today. |
| `crm.meetings_booked`  | CRM           | Activities, Activity Type Meeting Scheduled, in the period.                                                      |
| `crm.new_engagements`  | CRM           | Engagements created in the period, by Engagement Type.                                                           |
| `crm.pipeline`         | CRM           | Every Engagement by current Status. Cumulative, not period.                                                      |
| `team.meetings_held`   | Team Activity | Per tracked person: Activity Type Meeting, in the period.                                                        |
| `team.demos_booked`    | Team Activity | Per tracked person: the brief's demo rule (default: Meeting Scheduled whose Notes contain "demo").               |
| `team.emails_sent`     | Team Activity | Per tracked person: Activity Type Email Sent or Email Reply, Direction Outbound, in the period.                  |

The CRM rows use the field contract in [`../sales-pipeline/SCHEMA.md`](../sales-pipeline/SCHEMA.md) (Activities: Activity Type, Direction, Classification, Actor, Date, Notes; Engagements: Engagement Type, Status). Every CRM and Team Activity count excludes Classification `{{EXCLUDED_CLASSIFICATIONS | Noise, Internal, System}}`. If the company's CRM is not that kit, the brief's CRM map says which of its fields play each part.

## Report Runs

One row per run that did work. A deck-routine run that stops on the deck-day guard writes no row: it updates Next Deck Day and Last Run Summary only. Primary field **Run**.

| Field                   | Type          | Notes                                                                                                                        |
| ----------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Run                     | single line   | `<report> · <YYYY-MM-DD HH:mm>` in `{{TIME_ZONE}}`.                                                                          |
| Run Number              | autonumber    |                                                                                                                              |
| Run ID                  | formula       | `"{{ID_PREFIX}}-R" & RIGHT("0000" & {Run Number}, 4)`                                                                        |
| Report                  | single select |                                                                                                                              |
| Started At              | date-time     |                                                                                                                              |
| Mode                    | single select | The mode the run used. A Test Period run records Dry Run.                                                                    |
| Period                  | single line   | `YYYY-MM`.                                                                                                                   |
| Period Label            | single line   | As printed, for example `March 2027 (through Mar 25)`.                                                                       |
| Outcome                 | single select | OK, Partial (some blocks skipped), Already Delivered, Stopped (a gate or guard), Error.                                      |
| Snapshot Captured At    | date-time     | From the snapshot row. Blank if it was not read.                                                                             |
| Snapshot Schema Version | number        |                                                                                                                              |
| Sources Used            | multi select  |                                                                                                                              |
| Sources Missing         | multi select  | Every source the run needed and could not read.                                                                              |
| Blocks Built            | number        | Pulse: message blocks. Deck: slides.                                                                                         |
| Blocks Skipped          | long text     | One line each: `<block or slide> — <reason>`.                                                                                |
| Deck File URL           | url           | Deck only.                                                                                                                   |
| Payload File URL        | url           | Deck only. The frozen `payload.json` the deck was built from. It is the audit record of what the deck said.                  |
| Message Link            | url           | The chat message's permalink.                                                                                                |
| Fallback Message        | long text     | Written only when the chat post failed: the exact message the owner should have received. The fallback automation emails it. |
| Summary                 | long text     | Counts, sources, links. No metric values, no names, no message text.                                                         |
| Error Message           | long text     | One line. No tokens, headers, or file contents.                                                                              |
| Error Status            | single select | New on an Error row. People move it on.                                                                                      |

## Highlights (optional)

Things a person wants in the next deck that no system records: a hire, a signed partnership, a board resolution, a customer story. Kept apart from computed data so a recompute never overwrites what a person typed. Primary field **Title**.

| Field        | Type          | Notes                                                                                             |
| ------------ | ------------- | ------------------------------------------------------------------------------------------------- |
| Title        | single line   | One line, stated as a fact.                                                                       |
| Detail       | long text     | One or two sentences. No figure unless it comes from a source in Link.                            |
| Category     | single select |                                                                                                   |
| Occurred On  | date          |                                                                                                   |
| Period       | single line   | `YYYY-MM`. The deck period it belongs to.                                                         |
| Include      | checkbox      | Default on. Untick to keep it out without deleting it.                                            |
| Source Link  | url           | The document, announcement, or record that shows it.                                              |
| Added By     | single line   |                                                                                                   |
| Highlight ID | formula       | `"{{ID_PREFIX}}-H" & RIGHT("000" & {Highlight Number}, 3)` over an autonumber `Highlight Number`. |

## Views

The API cannot create filtered views. They are in [`MANUAL.md`](MANUAL.md).

| Table              | View       | Filter and sort                                                            |
| ------------------ | ---------- | -------------------------------------------------------------------------- |
| Report Settings    | All        | none. **Do not filter, do not delete** (automations watch it).             |
| Metric Definitions | Active     | Metric Status = Active. Grouped by Block, sorted by Order.                 |
| Metric Definitions | Pulse      | Active and In Pulse. Sorted by Block, then Order.                          |
| Metric Definitions | Deck       | Active and In Deck. Sorted by Block, then Order. Headline shown.           |
| Metric Definitions | History    | Metric Status = Retired. Sorted by Key, then Version.                      |
| Report Runs        | All        | none. **Do not filter, do not delete** (automations watch it).             |
| Report Runs        | Recent     | Started At, newest first.                                                  |
| Report Runs        | Deliveries | Report = Stakeholder Deck, Outcome OK or Partial, Mode Live. Newest first. |
| Report Runs        | Errors     | Outcome = Error, Error Status New or Investigating.                        |
| Highlights         | Upcoming   | Include on. Sorted by Period, then Occurred On, newest first.              |

## Interface

**Reporting.** Drafted by the agent where the tool allows; finished and shared by a person.

| Page       | Shows                                               | People do                                                   |
| ---------- | --------------------------------------------------- | ----------------------------------------------------------- |
| Settings   | Both Report Settings rows                           | Change Mode, Destination, meeting fields, overrides (owner) |
| Metrics    | Metric Definitions, Active view, with a detail page | Read definitions; propose a change (a new Proposed row)     |
| Runs       | Report Runs, Recent and Errors                      | Open the deck and payload links; triage errors              |
| Highlights | Highlights, Upcoming (if on)                        | Add an item for the next deck; untick Include               |

## Automations

Created as drafts. A person turns each on after a test record proves it.

| Automation          | Trigger                                                             | Action                                                                             | Default |
| ------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------- |
| Error email         | Report Runs record created with Outcome = Error                     | Email `{{OWNER_EMAIL}}`: Run ID, Report, Error Message. Nothing else.              | on      |
| Fallback delivery   | Report Runs → Fallback Message becomes non-empty                    | Email `{{OWNER_EMAIL}}` the Fallback Message, subject `<Report> · <Period Label>`. | on      |
| Settings change log | Mode, Destination, or External Recipients changes on a settings row | Email `{{OWNER_EMAIL}}` the report, the field, its new value, and Changed By.      | on      |
