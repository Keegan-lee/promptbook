# SPINE — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the owner and the team. Replace every `{{…}}`, drop sections for features the brief turned off, and add the live links.

The **SPINE** is {{COMPANY}}'s one canonical "current state" document: current numbers, live deadlines, open items, who is who now. Every agent and every person reads it first.

- Current SPINE: open the [pointer record]({{URL_POINTER_RECORD}}) and follow its Drive Link. Do not bookmark the file itself: a new file replaces it every week.
- Data Room: [{{COMPANY}} Data Room]({{URL_DATA_ROOM}}).
- Agent protocol: [{{AGENT_PROTOCOL_NAME}}]({{URL_AGENT_PROTOCOL}}).

## Your week

| When                      | What arrives             | What you do                                                                                         |
| ------------------------- | ------------------------ | --------------------------------------------------------------------------------------------------- |
| {{FILING_CRON}}           | Drive Filing report (DM) | Work the manual tasks. Carry held files to sister entities' drives. Glance at Data Room candidates. |
| {{SYNC_CRON}}             | SPINE Sync DM            | Answer NEEDS YOU first. Skim what was applied. Open the new SPINE if anything big moved.            |
| Any time                  | An urgent DM (and push)  | A conflict touching money, a near deadline, or a third-party commitment. Act today.                 |
| When something is settled | —                        | Record a Decision, or log an Observation. The next sync picks it up.                                |

A quiet week sends two lines and writes no new SPINE. That is correct.

## How the SPINE changes

Only `CS · SPINE Sync` and {{OWNER_NAME}} write it. Every week the sync:

1. Gathers everything that changed since the SPINE's Last verified date (never less than {{GATHER_FLOOR_DAYS}} days).
2. Classifies each change: **REPLACE** (a line is wrong), **DELETE** (no longer current), **PROMOTE** (a decision is now in effect), **EXTRACT** (true but settled: moved to its own document), **RECONCILE** (a real conflict it will not settle), **NOTE** (unverified or minor).
3. Writes a new file `SPINE_<date>`, reads it back, moves the old one to the archive, and moves the pointer last.
4. Writes an audit record to Observations and DMs you.

The SPINE stays under {{WORD_CEILING}} words by extraction, never by cutting. A line that moved out is replaced by one pointer line naming its Article.

## Settling a NEEDS YOU item

Each item is a line in the SPINE ending `[UNRECONCILED <date> — see Observation <id>]`. The old wording stays until you settle it. Two ways:

1. **Decide it.** Create a Decision (Reasoning and Rejected alternatives filled), then open the reconcile Observation and set **Promoted to** that Decision. The next sync promotes it and removes the marker.
2. **Fix the source.** If one system was simply wrong, correct that system. The next sync sees them agree and removes the marker.

Either way, nothing changes until the next sync. To see everything open, use Observations → **Open Reconciles**.

## Standing facts

Standing facts are things you have settled so no agent asks again. They live in {{STANDING_FACTS_HOME}}. Only you add, edit, or remove one. Write one dated line each: `<YYYY-MM-DD> · <fact> · <evidence>`. The sync carries them word for word and never re-raises them. If new evidence later contradicts one, the DM lists it under "Standing facts to review"; the fact stays until you change it.

## Editing the SPINE by hand

You may edit the current SPINE file directly. Keep the header lines, the `END OF SPINE` line, and every marker you are not settling. Add one line to the corrections log: `Owner edit <YYYY-MM-DD>: <what changed>`. The next sync takes your text as its starting point. Never create a second file in the SPINE folder yourself: the folder must hold exactly one file, the one the pointer names.

## The Data Room

| Table                              | Use it to                                                                                                                                                                |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Articles                           | Register a document: the file in the drive, the row here. Set Category, Section, Entity, Status. Set Execution to `Signed` only when a signed copy is on file.           |
| Decisions                          | Record why the company chose something, and what it rejected. Write it as a resolution. Fill Rejected alternatives in the same sitting: it stops the question reopening. |
| Observations                       | Log a fact, a doubt, or a question in one paragraph. Set Author and Source. Stop. No tags.                                                                               |
| {{#if BOARD_MINUTES}}Board Minutes | One meeting or resolution. Only `Executed` and `Filed` count as enacted. Link the signed Article and each Decision it enacted.{{/if}}                                    |
| Sections                           | The internal shelves for Articles.                                                                                                                                       |
| Company State Settings             | The switches for both routines.                                                                                                                                          |

- **Decisions hold the why, not the numbers.** Write "see SPINE §5" rather than a figure.
- **Decision status:** `Locked` (do not relitigate; change course only with a new decision), `Provisional`, `Superseded` (link the replacement), `Reversed`.
- Agent-made rows say so: Observations with Author `Agent`, Articles with Registered By filled. Rows with State Role `SPINE Extract` wait at `review` until you confirm them.
- Never paste into the Data Room: canonical numbers, text copied from the SPINE, transcripts or mail bodies, secrets, personal data the record does not need.
- {{#each OTHER_HOMES}}{{WHAT}} goes in {{WHERE}}, not here. {{/each}}

## The settings rows

Open Company State Settings. One row per routine. Never add a second row for the same routine.

| Mode    | SPINE Sync                                                                                                  | Drive Filing                     |
| ------- | ----------------------------------------------------------------------------------------------------------- | -------------------------------- |
| Paused  | Does nothing.                                                                                               | Does nothing.                    |
| Dry Run | Writes one audit record with every proposed change and the composed text. Writes no file, moves no pointer. | Plans every move. Moves nothing. |
| Live    | Full run.                                                                                                   | Moves files within Batch Limit.  |

If Mode dropped to Paused by itself, **Kill Reason** says why (two failed writes in a row). Fix the cause, then raise it again.

{{#if DRIVE_FILING}}

## Filing

- Drop files into **{{DRIVE_INBOX}}**. Drive Filing moves them every {{FILING_CRON}} into the right department folder{{#if RENAME_ON_FILE}} and renames them `{{NAMING_CONVENTION}}`{{/if}}.
- It never deletes, copies, or edits a file, and never touches the SPINE folders{{#each PROTECTED_FOLDERS}}, {{NAME}}{{/each}}.
- Anything it cannot place stays in the inbox and appears on the manual-task list with a reason.
- Files changed in the last {{SETTLE_HOURS}} hours wait a week.
  {{#if SISTER_ENTITY_ROUTING}}- Sister entities' files land in their hold folders inside the inbox. Carry them to that entity's drive.{{/if}}
- Data Room candidates are listed{{#if REGISTER_CANDIDATES}} and registered as Articles at `not started`{{/if}}. Set Execution yourself.

The taxonomy:

{{DEPARTMENT_TAXONOMY}}
{{/if}}

## Rolling back a version

Every Live run's audit record (Observations → Sync Log) names the new file, the archived prior file, and every change with its old and new text.

- **Undo one change:** edit the current SPINE by hand (above), using the old text from the audit record.
- **Undo a whole version:**
  1. Move the bad SPINE file from the SPINE folder into the archive.
  2. Move the prior file (named in the audit record) from the archive back into the SPINE folder.
  3. On the pointer record, set Drive Link to that file, and Name and Description to its version.
  4. Log an Observation: `Rollback <date>: <bad version> → <restored version>, because …`.
  5. If the routine caused it, set its Mode to `Dry Run` until the cause is fixed.

Never trash either file.

## When something looks wrong

| Symptom                                 | Look at                                                                                              |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| No DM on {{SYNC_CRON}}                  | The routine's last run in the scheduler, then Last Run Summary on its settings row                   |
| Two files in the SPINE folder           | A run failed mid-write. The next sync archives the orphan. Do not delete either.                     |
| The pointer opens a file in the archive | A run failed before moving the pointer. The next sync finishes it; or follow "Undo a whole version". |
| A figure looks wrong                    | The audit record for that change: which source it came from. Fix the source, or log an Observation.  |
| The same question keeps coming back     | Settle it, or make it a standing fact.                                                               |
| "Not gathered" keeps naming a source    | That connector lost access. Restore it, or remove the source from the brief and re-fill the routine. |
| A file vanished from the inbox          | Drive Filing's last report: it names the destination. Nothing is ever deleted.                       |
