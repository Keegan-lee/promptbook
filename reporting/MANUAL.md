# Reporting — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past an authentication wall on the person's behalf.

## Connections (before the build)

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the reporting base will live, and read access to the CRM base.
2. **Snapshot read.** Give the routines' database connector a **read-only** role that can read the snapshot table and nothing it does not need. Never an admin key, and never a role every product user holds. For the file alternative, give the storage connector read access to the snapshot folder only.
3. **File storage.** Connect it with access to the deck folder, the outline, the logos, and the company-state document.
4. **Chat.** Connect the chat tool. Note the owner's DM target and the pulse destination.
5. **Optional:** the decisions log (read only), mail drafts (only if outside recipients are named), and each run-time source (payment processor, spreadsheet, analytics) with a read-only key. Put every key in the connector UI or the secret store, never in chat.

## The snapshot (with a codebase)

1. **Production change.** Apply the snapshot table and its access settings to production yourself (or the person the brief names). The developer agent prepares it and does not apply it.
2. **Deploy** the producer the developer agent built, and confirm its nightly schedule.
3. **Backfill** last month and the current month with the manual run.
4. **Verify** from the routines' connector: the read query returns one row for the current month. Zero rows means the connector reads as the wrong role. Fix the role; never grant read access to every signed-in user.
5. **Allowlist** the logo host and the storage download domain in the routine environment if the sandbox blocks them.

## After the base is built

1. **Time zones.** Confirm every date-time field (Started At, Snapshot Captured At, Last Run At, Changed At) displays `{{TIME_ZONE}}`, 24-hour. The API cannot change a date-time field's zone after it is created.
2. **Select hygiene.** Look for options with trailing spaces or near-duplicates on Report, Mode, Block, Source, and Outcome. Routines match these exactly.
3. **Views.** Create every view in [`SCHEMA.md`](SCHEMA.md) → Views. Report Settings → All and Report Runs → All stay unfiltered: automations watch them.
4. **Interface: Reporting.** Finish the pages the agent drafted: Settings (editable by the owner only), Metrics, Runs (with the Deck File URL and Payload File URL as links), Highlights (if on). Share it with the people the brief names, and nobody else: the base holds revenue and per-person activity.
5. **Automations.** Turn each on only after a test proves it, then delete the test record:
   - **Error email:** create a Report Runs row with Outcome Error. The owner gets one email with the Run ID, the report, and the one-line error.
   - **Fallback delivery:** fill Fallback Message on a test row. The owner gets the message by email.
   - **Settings change log:** change Mode on a settings row. The owner gets one email.
6. **Settings rows.** The owner confirms Mode = Paused on both rows, Destination on both, Tracked People on the Pulse row, and the meeting fields, Outline URL, and Deck Folder URL on the Deck row. External Recipients stays empty unless the brief named recipients.

## The deck

1. **Folder.** Create the deck folder and its `Dry runs` subfolder. Set their sharing yourself. The routine never changes sharing.
2. **Logos.** Put the light-background and dark-background logo files where the outline says. Check by eye which one is truly white.
3. **Outline.** Read the outline, edit it, and approve v1.0. It must end with `END OF DECK OUTLINE`.
4. **Test deck.** Open the test deck for last month in `Dry runs`. Check every number against its source with the build agent (kickoff Phase 6). Approve the look.
5. **Outside recipients** (only if the brief names any): before you send the first draft, share the deck folder with them or attach the file by hand. The draft never carries a public link.

## Routines

1. **Environment.** The deck routine needs code execution with the generator's runtime, Pillow, LibreOffice, and poppler. If the cloud sandbox lacks them, run it as a desktop scheduled task on a machine that is awake at the run time.
2. **Tool policies.** Confirm each routine's connector policies match [`KICKOFF.md`](KICKOFF.md) Phase 5: no database writes, no CRM writes, no file delete, move, overwrite, or sharing, no mail send.
3. **Schedules.** Confirm each cron is converted from `{{TIME_ZONE}}` to the scheduler's zone, and check it again after a daylight-saving change.
4. **Enable** each routine only after kickoff Phase 6, with Mode Dry Run.

## The owner's switches

These are the owner's alone. No routine moves them up.

| Switch                       | Where                      | When                                                            |
| ---------------------------- | -------------------------- | --------------------------------------------------------------- |
| Pulse Mode: Paused → Dry Run | Report Settings, Pulse row | After the Phase 6 reconciliation matches                        |
| Pulse Mode: Dry Run → Live   | Pulse row                  | After one real pulse in your DM looked right                    |
| Deck Mode: Paused → Dry Run  | Deck row                   | After the test deck for last month is approved                  |
| Deck Mode: Dry Run → Live    | Deck row                   | After one real deck day in Dry Run                              |
| Meeting Date Override        | Deck row                   | Any month the meeting moves off the rule                        |
| Build Now                    | Deck row                   | After a missed deck day, or to rebuild a delivered period       |
| Test Period                  | either row                 | To re-run a past month as a test                                |
| External Recipients          | Deck row, and the brief    | Only with a matching change to the brief and a refilled routine |

## Checks a person does once

- Report Settings → Deck row → Next Deck Day shows the day before your next meeting (by the lead days), not a day after it.
- A `[TEST]` pulse for last month matches the CRM, filtered to the same month in `{{TIME_ZONE}}`, person by person.
- The test deck's appendix lists a source and an "as of" time for every number.
- A metric you made unreadable on purpose (a wrong path in a test row) shows "not measured", never 0.
- Report Runs → Errors is empty after verification, and no test record is left in the base.
