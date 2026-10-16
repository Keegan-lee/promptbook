# CP · Scheduler

| Setting | Value                                                                                       |
| ------- | ------------------------------------------------------------------------------------------- |
| Trigger | Every `{{SCHEDULER_INTERVAL \| 30}}` minutes                                                |
| Reads   | Settings, Platform Posts (with its Content lookups), Platform Rules                         |
| Writes  | Platform Posts → Scheduled DateTime and Notes; Content → Status (Approved → Scheduled only) |
| Modes   | Paused, Dry Run, Live. Drafts behaves as Dry Run.                                           |

## Purpose

Give every fully approved post a time: the content's Launch At when it has one, otherwise the channel's next free slot. Distribution sends at that time.

## Rules (never break)

- **Fill blanks only.** Never overwrite a Scheduled DateTime that is already set, whether a person typed it, Launch At filled it, or an earlier run did. Re-read the post right before writing; if the field is no longer blank, skip it.
- **Launch At wins** over channel slots.
- **Priority first:** posts whose Content Priority is "Priority" are placed before "Backlog" (or empty).
- **Slots come from Platform Rules, in local time:** Posting Days and **Posting Time (Local)** in `{{TIME_ZONE}}`. Never read a UTC date-time field for a slot.
- **One post per channel per local day.** Two posts on one channel the same day compete with each other.
- The scheduler never sends and never changes Approval.

## Candidates

Platform Posts where all of these hold (the approval half of the gate):

- Scheduled DateTime is blank;
- Upload Status = Pending;
- Approval = Approved, Copy Approved and Creative Approved ticked, Approved By not empty (`LEN({Approved By} & '') > 0`);
- Content Status is Approved or Scheduled;
- Content Is Held = 0;
- Creative Ready = 1 (no creative needed, or every linked asset approved by a person and a Media URL present), so the calendar never fills with posts that are waiting for a design;
- the channel's Platform Rules row is active.

Order: Content Priority (Priority first), then Content Launch At set before unset, then Approved At oldest first. Batch: up to `{{SCHEDULER_BATCH | 50}}` posts per run.

## Slots

For a channel's rule:

1. **Days.** Posting Days if set. Otherwise derive from Frequency: Daily = every day; 3x Weekly = Mon, Wed, Fri; 2x Weekly = Tue, Thu; Weekly = Tue.
2. **Holidays.** When `{{SKIP_HOLIDAYS | yes}}` is yes, drop dates on the brief's holiday calendar.
3. **Candidate times.** For each remaining date in the next 28 days, the slot is that date at Posting Time (Local) in `{{TIME_ZONE}}`. Convert it to UTC with that date's offset (daylight saving changes the offset).
4. **Lead time.** Drop slots earlier than now plus `{{MIN_LEAD_MINUTES | 60}}` minutes.
5. **Taken.** A slot is taken when any post on the same Platform has a Scheduled DateTime on the same local date and Upload Status is Pending, Sent, or Posted. Count the slots this run has already assigned.

## Steps

1. Read the Settings row. Paused: stop.
2. Load active Platform Rules rows. Load every post with a Scheduled DateTime in the next 28 days (for the taken check).
3. Load the candidates in order.
4. For each candidate:
   - **Content Launch At is set:**
     - in the future: the time is Launch At, even if another post is on that channel that day (a launch is deliberate);
     - in the past: do not schedule. Notes line, once: `Launch At has passed; set a new Launch At or clear it`.
   - **Otherwise:** the first free slot on that channel. None in 28 days: Notes line, once: `No free slot in 28 days`.
   - **Dry Run:** Notes line, once: `DRY RUN <date>: would schedule for <local date and time>`. Mark the slot taken in memory. Write nothing else.
   - **Live:** re-read the post. Still blank and still a candidate: write Scheduled DateTime (UTC). Mark the slot taken.
5. For each Content row that got its first scheduled post this run and has Status = Approved: set Status = Scheduled.
6. Write the Last Run Summary: posts scheduled per channel, posts waiting with a reason, record ids.

If Launch At changes after posts were scheduled, a person clears the affected posts' Scheduled DateTime and the next run refills them from the new Launch At. The scheduler does not chase changes on its own, because it cannot tell a deliberate hand-set time from a stale one.

## n8n notes

- Schedule Trigger every 30 minutes; set the workflow's time zone to `{{TIME_ZONE}}`.
- Do the slot maths in one Code node with a time-zone library (`luxon` is available in n8n Code nodes): `DateTime.fromISO(date + 'T' + time, { zone: '{{TIME_ZONE}}' }).toUTC().toISO()`.
- Airtable search returns values under `json.fields`. Lookups come back as arrays: take the first value.

## Test

With the Settings row on Dry Run, approve a `[TEST]` post on one channel in the UI. Within 30 minutes a `DRY RUN` line names the next slot. Set Launch At on its Content row to tomorrow and clear the line: the next run names Launch At instead. Move to Live on a test channel only, confirm the time is written in UTC and displays correctly in the local zone, then clear the test post.
