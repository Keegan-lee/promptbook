# Reporting — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the owner. Replace every `{{…}}`, drop sections for features the brief turned off, add the live links, and delete this note. Keep it short enough to read on a phone.

Live base: [{{BASE_NAME}}]({{URL_BASE}}). Interface: [Reporting]({{URL_INTERFACE}}). Deck folder: [{{DECK_FOLDER_NAME}}]({{URL_DECK_FOLDER}}). Outline: [{{DECK_AUDIENCE}} deck outline]({{URL_OUTLINE}}). Brief: [Reporting brief]({{URL_BRIEF}}).

## Your month

| When                               | What arrives                                                  | What you do                                                                                              |
| ---------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| {{PULSE_SCHEDULE_TEXT}}            | The weekly pulse ({{PULSE_DESTINATION_TEXT}})                 | Read it. If a block is missing, the footnote says which source; see "When something looks wrong".        |
| Before deck day                    | Nothing                                                       | Add anything no system records to Highlights. Move the meeting with Meeting Date Override if it changed. |
| Deck day ({{DECK_DAY_TEXT}})       | "Deck ready" message, and owner checks if any                 | Open the deck. Read the owner checks first. Edit a copy if you want; never the delivered file.           |
| {{#if MAIL_DRAFTS}}Deck day{{/if}} | {{#if MAIL_DRAFTS}}An email draft to {{DECK_AUDIENCE}}{{/if}} | {{#if MAIL_DRAFTS}}Attach or share the deck, then send it yourself. Nothing is sent for you.{{/if}}      |
| Meeting day                        | Nothing                                                       | Present.                                                                                                 |

The next deck day is always on the Deck row → **Next Deck Day**. Check it after you change a meeting.

## What each table is for

| Table              | You use it to                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| Report Settings    | Switch each report on or off, change where it posts, move a meeting, rebuild a deck, test a past month. |
| Metric Definitions | See exactly what every number means and where it comes from. Change a definition (below).               |
| Report Runs        | See every run: what it read, what it left out and why, the deck, and the frozen payload behind it.      |
| Highlights         | Add a hire, a partnership, a board decision, or a customer story for the next deck.                     |

## Modes

| Mode    | Pulse                                  | Deck                                                                          |
| ------- | -------------------------------------- | ----------------------------------------------------------------------------- |
| Paused  | Nothing.                               | Nothing.                                                                      |
| Dry Run | Posts to your DM only, marked DRY RUN. | On deck day, builds into `Dry runs` and DMs you only.                         |
| Live    | Posts to {{PULSE_DESTINATION_TEXT}}.   | On deck day, saves to the deck folder and posts to {{DECK_DESTINATION_TEXT}}. |

If a run stopped on a guard, **Kill Reason** says why. Fix the cause; the next run picks it up.

## Reading the pulse

- **Growth** lines: `<metric>: <this month> this month · <all time> total`, from the metrics snapshot. "not measured" means the source could not be read for that number. It is not zero.
- **Team activity**: per person, month to date, from the CRM. Noise, internal, and system activity are left out. An activity with no Actor (the CRM's nightly reconciliation has not linked it yet) does not count, so early-week numbers can read low.
- **Footnotes**: the demo rule, when the snapshot was captured, and every block left out with its reason.
- In the first week of a month the numbers are small. {{#if PREVIOUS_MONTH_LINE}}The pulse adds a line with last month's numbers.{{/if}}

## Reading the deck

- Every number traces to a source. The **appendix** slide lists each metric's definition, source, and "as of" time.
- A slide is left out when its source was missing or empty. The deck message lists what was left out and why.
- Period labels say "(through <date>)" when the month is still in progress, and "(full month)" when it is closed.
- **Owner checks** are for you only: a figure in the company-state document that disagrees with the snapshot (the deck used the snapshot), an outside recipient not in the brief, a stale snapshot.
- The `payload - <period>.json` file beside each deck is the record of exactly what it said. Keep it.

## Changing things

| You want to                                  | Do this                                                                                                                                                                                                    |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Change what a number means                   | Change the brief first. Then add a new Metric Definitions row with the same Key, Version + 1, and Effective From; retire the old row. Ask the agent to refill the routines. The deck footnotes the change. |
| Show or hide a number                        | Tick or untick In Pulse, In Deck, or Headline. Effective on the next run.                                                                                                                                  |
| Rename a label                               | Edit Metric. Keep its Wording Rule.                                                                                                                                                                        |
| Add or drop a slide, change the look         | Edit the outline, raise its version. Ask the agent if a new slide needs new data.                                                                                                                          |
| Move this month's meeting                    | Set Meeting Date Override on the Deck row. Check Next Deck Day after the next run.                                                                                                                         |
| Rebuild a deck, or recover a missed deck day | Tick Build Now. The next run builds the nearest meeting's period and saves `… v2`; the delivered file is never overwritten.                                                                                |
| Re-run a past month as a test                | Set Test Period (`YYYY-MM`) on either row. It always delivers as Dry Run.                                                                                                                                  |
| Add or remove a person in the team table     | Edit Tracked People: `Display Name · <CRM Team record id>`, one per line.                                                                                                                                  |
| Add an outside recipient                     | Change the brief, then External Recipients, then ask the agent to refill the deck routine. An address in only one place is never drafted.                                                                  |

## When something looks wrong

| Symptom                             | Look at                                                                                                                                            |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| No pulse this week                  | Pulse row: Mode, Last Run Summary. Report Runs → Recent: a Fallback Message means the chat post failed. Then the scheduler's run history.          |
| Growth block missing                | The footnote. "0 rows" means the snapshot connector reads as the wrong role. "file missing" or an old capture time means the producer did not run. |
| A number disagrees with a dashboard | The window (month to date in {{TIME_ZONE}}), the definition's Version and Effective From, and the snapshot's capture time.                         |
| Team table reads low                | Whether the CRM's reconciliation has run (Actor blank, Classification Unclassified), and Tracked People ids.                                       |
| Deck did not arrive                 | Deck row: Mode, Next Deck Day, Meeting Rule, Meeting Date Override, Last Delivered Period, Kill Reason. Then Report Runs → Errors.                 |
| A slide is missing                  | Report Runs → the deck run → Blocks Skipped.                                                                                                       |
| A logo shows as text                | Sources Missing lists Logos: the file moved, or the host is blocked in the routine's network allowlist.                                            |
| A pulse and a deck disagree         | The pulse is month to date on its run day; the deck reports {{REPORT_PERIOD_TEXT}} on deck day. Same definitions, different windows.               |
