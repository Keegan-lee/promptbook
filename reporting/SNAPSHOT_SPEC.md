# Metrics snapshot — producer spec

The contract between a **producer** in the company's codebase and the **consumers** in this kit (`RP · Metrics Pulse` and `RP · Stakeholder Deck`). Hand this file, the reporting brief, and the Metric Definitions table to a developer agent that works in the codebase, and say: **"Read `SNAPSHOT_SPEC.md` and build the metrics snapshot producer."**

The producer computes every product number once a night and writes it to **one read-only place**: a table (or view) with one self-contained row per month, or one JSON file per month. The routines read that one place. They never query live product tables, and they never type a number.

No codebase or no database? Skip this file. The reporting kit's run-time mode computes each metric when the routine runs, from the sources the interview names (a payment processor, a spreadsheet, an analytics tool), and labels every number with its source and date. See [`KICKOFF.md`](KICKOFF.md), Phase 3.

## Interview (developer agent)

Ask only what the reporting brief does not already answer. Record answers in a short `SNAPSHOT_BRIEF` section of the brief.

1. Which metrics? Read the Metric Definitions rows whose Source is `Snapshot`. Each row's Key, Definition, Formula, Period Path, and Cumulative Path is the spec for one number. A metric with no agreed definition is not built; it goes back to the owner.
2. Which stores hold the data (relational database, document store, the auth provider's user list, the payment processor's records)? Can one scheduled job reach all of them?
3. Where do scheduled jobs live in this codebase (cloud functions, a cron service, a job runner)? Put the producer beside them. One cron runtime is easier to reason about than two.
4. Which role will the routines use to read the snapshot (the connector's database role, a storage account)? It must be read-only and must not be a role every signed-in product user holds.
5. The home time zone (`{{TIME_ZONE}}`). Month boundaries use it.
6. Where the brand's design tokens and logo files live (a design-system package, CSS variables, a brand folder).
7. The earliest time any routine reads the snapshot. The producer finishes at least 90 minutes before it.

## Design decisions

These held up in production. Keep them unless the brief gives a reason.

| Decision                                   | Why                                                                                                                                                                                                                                                                                                                   |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| One self-contained row (or file) per month | The consumer makes **one** read and gets everything. A headless job in a restricted sandbox should have the fewest moving parts possible.                                                                                                                                                                             |
| A JSON payload, not one column per metric  | Adding a metric needs no migration, only a schema-version bump. The row is a frozen record: a definition that changes later does not rewrite what the company already told its stakeholders.                                                                                                                          |
| Nightly, not monthly                       | A monthly job has one chance and fails on the morning of the meeting. Nightly, a failed run degrades to yesterday's numbers, and `captured_at` makes the staleness visible.                                                                                                                                           |
| Human-authored notes in their own table    | Hires, board resolutions, and partnerships are not derivable from code. Keeping them apart means a recompute never overwrites what a person typed.                                                                                                                                                                    |
| Server-only access                         | Revenue and user counts are not public data. The snapshot is readable by the reporting role only, never by every signed-in user.                                                                                                                                                                                      |
| `null` means "could not measure"           | A sub-source that fails (a document store, a third-party API) writes `null` and names the key in `unmeasured`. It never writes `0`. Zero means "measured, and none".                                                                                                                                                  |
| Zero-valued sections still ship            | A block that is all zeros today (orders before the first order) stays in the payload, so the deck's shape does not change the month the first one lands.                                                                                                                                                              |
| Distinct names for distinct things         | People who signed up, people who hold a role, and things people created (listings, locations, stores) are three metrics with three names. One label for two of them is a bug that ships to a board.                                                                                                                   |
| Month boundaries in the home time zone     | The routines filter CRM activity by month in `{{TIME_ZONE}}`. A snapshot anchored to UTC disagrees with the CRM for the hours either side of midnight on the first. Use the home zone and write it into the payload. If the codebase must use UTC, write `"time_zone": "UTC"` and the routines footnote the mismatch. |
| Dormant-safe                               | If the snapshot table does not exist yet (the migration is not applied), the producer computes, logs a warning, and skips the write. Deploying code before the migration breaks nothing.                                                                                                                              |
| Tolerate missing optional tables           | Newer tables are read through a wrapper that turns "relation does not exist" into a `null` for that key and a log line, not a failed run.                                                                                                                                                                             |

## 1. Read path

One query, read-only:

```sql
select period_month, schema_version, payload, period_start, period_end, captured_at
  from {{SNAPSHOT_TABLE | reporting.metrics_snapshot}}
 order by period_month desc
 limit 1;
```

To pin a month:

```sql
select payload, captured_at, schema_version
  from {{SNAPSHOT_TABLE | reporting.metrics_snapshot}}
 where period_month = date 'YYYY-MM-01';
```

| Column           | Type         | Notes                                                                      |
| ---------------- | ------------ | -------------------------------------------------------------------------- |
| `period_month`   | date         | Primary key. First day of the reported month.                              |
| `schema_version` | integer      | Payload contract version. Starts at `1`.                                   |
| `payload`        | json / jsonb | The whole dataset (§3).                                                    |
| `period_start`   | timestamptz  | Start of the measured window.                                              |
| `period_end`     | timestamptz  | `min(capture time, month end)`. Less than month end means a partial month. |
| `captured_at`    | timestamptz  | When the producer last wrote this row.                                     |

### Access

- Row-level security on, **no policies**, and every privilege revoked from the public and signed-in-user roles. The reporting role reads it because it bypasses row security or holds an explicit grant.
- Confirm the role the routine's connector actually uses. If a connector reads as the public role, the query returns **zero rows, not an error**. The routines treat zero rows as "snapshot unreadable", never as "no data".
- Never widen access to make a connector work. Granting select to every signed-in user would let any customer read company revenue. Fix the connector's role instead.
- A view over the table is fine if the codebase prefers views. Same access rules.

## 2. Reporting period

- `period_month` is the month being reported. The window is `[month start, min(now, month end))` in the payload's time zone.
- During the month, every nightly run **overwrites** the month's row in place (upsert on `period_month`). There is never more than one row per month.
- `payload.period.is_partial` is `true` while the month is in progress, and `period.end` is the capture time. The deck then says "March 2027 (through Mar 25)" instead of implying a closed month.
- **Closing run.** On the first run after a month ends, recompute that month once more with `is_partial: false` and `end` = month end, then never write that row again. Without this the closed month misses its last day.
- Re-running an old month on purpose (a backfill, a correction) is a manual run (§6) and is logged.

## 3. Payload shape (`schema_version: 1`)

A starting shape. The keys that ship are the Period Path and Cumulative Path values in Metric Definitions; drop blocks the business does not have and add the ones it does. Name a key after the thing it counts. Money is in minor units (cents) with an explicit currency.

```jsonc
{
  "schema_version": 1,

  "period": {
    "month": "2027-03-01",
    "label": "March 2027", // ready to print
    "start": "2027-03-01T00:00:00-05:00",
    "end": "2027-03-25T01:00:00-04:00",
    "is_partial": true,
    "time_zone": "America/New_York" // the zone the window was cut in
  },

  "growth": {
    "signups": { "new": 0, "cumulative": 0 }, // PEOPLE who created an account
    "customers": { "new": 0, "cumulative": 0 }, // people or organisations who paid at least once
    "<supply_unit>": { "new": 0, "cumulative": 0 } // THINGS people create or list; named for the thing
  },

  "revenue": {
    "currency": "USD", // the company's reporting currency
    "succeeded_minor": 0, // the one status that means money moved, in period
    "succeeded_count": 0,
    "refunded_minor": 0,
    "by_currency": [], // [{ "currency": "USD", "succeeded_minor": 0 }]; never summed across currencies
    "cumulative_succeeded_minor": 0, // all time, through period end
    "active_subscriptions": 0, // at period end; omit if there are no subscriptions
    "recurring_minor": null // monthly recurring revenue, only if the brief defines it
  },

  "marketplace": {
    // or "orders" for a business that is not a marketplace
    "new_orders": 0,
    "cumulative_orders": 0,
    "new_orders_by_status": {}, // { "confirmed": 3, "completed": 1 }; an absent status is zero
    "completed_orders": 0, // its own number, never folded into new_orders
    "inventory": {} // what can be bought or booked at period end: { "<unit>": 0 }
  },

  "usage": {
    "active_users": null, // the brief's definition of active; null until it is built
    "requests": 0,
    "error_requests": 0,
    "api_keys_issued": 0, // issued is not active
    "api_keys_with_traffic": 0,
    "measured_since": "2027-01-10" // collection start; early months read low, not wrong
  },

  "segments": {
    // optional: people by role or plan
    "by_role": {
      "new": { "<role>": 0 },
      "cumulative": { "<role>": 0 },
      "new_total": 0, // distinct people
      "cumulative_total": 0,
      "multi_count": true // a person with two roles counts in each; totals are distinct
    }
  },

  "community": {}, // optional: contributions, forum activity, saved items

  "notes": {
    // folded in from the notes table (§4)
    "administrative_changes": [{ "category": "hire", "title": "...", "detail": "...", "occurred_on": "2027-03-14" }],
    "notable": [{ "title": "...", "detail": "..." }]
  },

  "brand": {
    // §5
    "tokens": { "colors": {}, "semantic": {}, "fonts": {} },
    "assets": { "logo_light_bg": "<url or storage path>", "logo_dark_bg": "<url or storage path>" }
  },

  "provenance": {
    // one entry per top-level metric key
    "growth.signups": { "source": "<store and table>", "as_of": "2027-03-25T01:00:00-04:00" }
  },
  "unmeasured": [], // keys that are null because a sub-source failed this run

  "captured_at": "2027-03-25T01:00:00-04:00"
}
```

### Metric conventions

Apply these to every metric unless its Metric Definitions row says otherwise.

- **New** = created within `[period.start, period.end)`. **Cumulative** = existing at `period.end`.
- **Exclusions** are part of the definition: test accounts, internal users, merged duplicates, deleted or archived rows (say which), bot sign-ups if the product can tell. Write each exclusion into the Definition.
- **Revenue** counts only the status that means money moved (for example `succeeded`, `paid`, `settled`). Pending, authorised, and failed payments are not revenue. Credits, waivers, and internal transfers are not revenue; if the deck must mention them, they get their own key and their own label.
- **Roles** come from the system that grants them. If the role is a current attribute with no history, say so in the Definition: a person counts under the role they hold today, not the one they held at sign-up.
- **Measured-since.** A metric whose collection started recently carries `measured_since`. The routines footnote it.
- **No derived numbers the consumer could get wrong.** If the deck shows a rate (conversion, error rate), the producer emits the numerator and the denominator, and the routine divides. It never estimates either.

## 4. Human-authored notes

`{{NOTES_TABLE | reporting.metrics_snapshot_notes}}`: one row per month, `period_month` primary key, `administrative_changes` and `notable` as JSON arrays. The producer folds them into `payload.notes` on its next run.

- `administrative_changes[].category` is one of `hire`, `board`, `partnership`, `other`. Only `title` is required.
- A person fills it before deck day, or forces a recompute afterwards (§6).
- If the reporting kit's Highlights table is on, the deck routine reads Highlights directly and this table may stay empty. Do not maintain both by hand: the brief picks one.

## 5. Brand

Split by size, so the consumer never depends on a fetch it cannot afford to lose.

- **Tokens in the payload.** Colours (with semantic roles: primary, secondary, accent, background, surface, text, muted, danger), font families, and a spacing scale. They are a few kilobytes, arrive in the same read as the numbers, and drive every colour the deck uses.
- **Logo images by reference.** Too large to duplicate into a row every night. Put the storage path or HTTPS URL of each logo in `brand.assets`: one for light backgrounds and one for dark backgrounds. The host must be reachable from the routine's sandbox (add it to the network allowlist), and the files must be static, with no sign-in.
- **Verify the assets, not their names.** A file named "white" can be a coloured logo. Record in `brand.assets` which file is truly for dark backgrounds after looking at it.
- Public brand URLs move. The routine validates every logo it downloads (file signature, not just the extension) and falls back to a text wordmark. A missing logo never blocks a deck.
- Name web fonts rather than embedding font files.

## 6. Manual run

Ship an admin-only way to compute and upsert a month on demand: an authenticated endpoint, a CLI command, or a job trigger.

| Operation      | Does                                                                        |
| -------------- | --------------------------------------------------------------------------- |
| Run a month    | Compute and upsert `period_month` (idempotent). Default: the current month. |
| List snapshots | Months, schema versions, and capture times. No payloads.                    |
| Read one month | The full stored payload.                                                    |

Use it to backfill earlier months at go-live (the kickoff's dry run needs last month), to recompute after notes change, and to close a month by hand. Log who ran it. Never expose it without authentication.

## 7. Timing and failure

| Item     | Default                                                                                              |
| -------- | ---------------------------------------------------------------------------------------------------- |
| Producer | Nightly at `{{PRODUCER_TIME \| 01:00}}` in the producer's zone, with retries (3).                    |
| Margin   | At least 90 minutes before the earliest routine read.                                                |
| Failure  | A run that fails every retry leaves the previous night's row untouched. `captured_at` shows the age. |
| Alert    | A failed final retry alerts the owner through the codebase's own error channel.                      |

Keep the job cheap: a handful of aggregate queries, one pass over the user list if roles live in an auth provider, no per-row network calls.

## 8. File alternative

When the routines cannot reach a database but can reach file storage, write the same payload as files:

- `{{SNAPSHOT_FOLDER}}/snapshot-YYYY-MM.json`, one per month, plus `latest.json` (a copy of the current month).
- Write to a temporary name, then rename, so a reader never sees half a file.
- Same period, closing-run, `null`, and schema-version rules.
- Limit the folder's access to the owner and the routine's account. Never make it public.
- Some storage APIs report a size of 1 for native documents and do not create parent folders. Create folders one level at a time, and verify each write by reading the file back.

## 9. Extending

- **New metric:** add it to `growth` (or the right block), add its Metric Definitions row, bump `schema_version`. Old rows keep their version; the routines branch on it.
- **Never rename or repurpose a key.** Add a new key and retire the old Metric Definitions row. A renamed key silently empties a slide.
- **A definition change** is a new key or a version bump plus a Metric Definitions row with an Effective From. Never recompute closed months under a new definition without a note in `notes.notable` and a run log entry.
- **Reserved slots** (a changelog, a forecast) ship as `null` and render nothing until they are built.
- If a block is added before any row has been written to production, there is no need to bump the version: no consumer has seen the old shape.

## 10. Tests the producer ships

- An empty month: every count `0`, every section present.
- A partial month: `is_partial: true`, `end` before month end, label correct.
- The closing run: `is_partial: false`, the row not touched again.
- A month boundary in the home zone: an event one hour before local midnight on the last day counts in that month.
- Multi-currency revenue: `by_currency` populated, nothing summed across currencies.
- A failing optional sub-source: that key `null`, listed in `unmeasured`, the run still succeeds.
- A missing optional table: same as above.
- Read-back as the reporting role returns one row. Read-back as the public or signed-in-user role returns zero rows.
- The payload validates against a JSON schema generated from §3 and the Metric Definitions keys.

## Definition of done

- The table (or folder) exists with the access posture in §1, applied by the person who owns production changes.
- The producer is deployed, scheduled, and has written the current month and at least the previous full month.
- Every Metric Definitions row with Source `Snapshot` has a populated Period Path and Cumulative Path that resolve in the payload.
- A read through the routine's own connector returns the row.
- The manual run works and is admin-only.
- Tests in §10 pass.

## Hand back to the reporting kit

Give the build agent, in one message, for the brief's ID map:

1. The exact read query (or file path) and the role or account that runs it.
2. The schema version, the capture time of the latest row, and whether the previous month is closed.
3. Every key the payload fills, mapped to its Metric Definitions Key, and every metric you could not build, with why.
4. Where the logos are and which one is for dark backgrounds.
5. Anything a person must still do (apply a migration to production, add a host to an allowlist).
