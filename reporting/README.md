# Reporting kit

Two reports a company runs on its own numbers, without anyone typing a number from memory:

- **`RP · Metrics Pulse`** (weekly): one chat message to the owner with growth for the month to date and all time, and a per-person table of meetings held, demos booked, and emails sent from the CRM.
- **`RP · Stakeholder Deck`** (monthly): the advisor, board, or investor update. It runs on a schedule, acts only on deck day, builds the deck from the metrics snapshot, the CRM, the company-state document, and the decisions log, and delivers it to the shared drive and the owner.

It is company-agnostic. The interview in [`KICKOFF.md`](KICKOFF.md) makes it yours.

**To use it:** give your agent this folder plus [`../_shared/`](../_shared/), and say "Run the Reporting kickoff."

| File                                                           | What it is                                                                                                                         |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| [`KICKOFF.md`](KICKOFF.md)                                     | The prompt. Interview → brief → snapshot → base, outline, and generator → routines → dry run against last month → hand back.       |
| [`SCHEMA.md`](SCHEMA.md)                                       | The Airtable control panel: Report Settings, Metric Definitions, Report Runs, optional Highlights.                                 |
| [`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md)                         | The contract for a developer agent: one read-only row (or file) per month with every product number, refreshed nightly, versioned. |
| [`DECK_OUTLINE_TEMPLATE.md`](DECK_OUTLINE_TEMPLATE.md)         | The binding deck outline: slides, data keys, omit rules, number rules, narrative rules, what never goes in a deck, the look.       |
| [`routines/metrics-pulse.md`](routines/metrics-pulse.md)       | `RP · Metrics Pulse`: snapshot growth block plus CRM team-activity block, one message, one log row.                                |
| [`routines/stakeholder-deck.md`](routines/stakeholder-deck.md) | `RP · Stakeholder Deck`: deck-day guard, layered payload, narrative digest, generator, QA render, delivery.                        |
| [`MANUAL.md`](MANUAL.md)                                       | What a person must click, and the owner's switches.                                                                                |
| [`OPERATORS.md`](OPERATORS.md)                                 | The operator's manual template the agent fills in at the end.                                                                      |

## What the reports never do

- Type a number. Every number comes from the snapshot, a CRM count, or a source named in Metric Definitions, read during the run.
- Turn a missing source into a zero. The block or slide is left out and the report names the source. `null` prints as "not measured".
- Overwrite the snapshot's numbers. Later steps add keys; a check before rendering proves it.
- Copy a number from a previous deck.
- Put an internal section of the company-state document in a deck.
- Send email or change who can see a file. Default delivery is the shared drive and the owner's DM. Outside recipients get a draft that the owner sends.

## Build order and value

1. **Metric Definitions and the pulse.** Useful the first Monday. Runs in run-time mode if there is no snapshot yet.
2. **The snapshot producer** ([`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md)), built by a developer agent in the codebase. Both reports then read one frozen row a month.
3. **The deck.** Outline, generator, and the deck-day routine, proven against last month before it is enabled.

## Related kits

- **Sales Pipeline** ([`../sales-pipeline/`](../sales-pipeline/)): the team-activity block and the go-to-market slide read its Activities and Engagements field contract. With that kit installed, the reporting kickoff reuses its ID map. Without it, the interview maps the company's own CRM fields.
- **Content OS** ([`../content-os/`](../content-os/)): its brand pack supplies the deck's colours, fonts, and logos.
- **Blog** ([`../blog/`](../blog/)): [`SNAPSHOT_SPEC.md`](SNAPSHOT_SPEC.md) is handed to a developer agent the same way the blog's [`SPEC.md`](../blog/SPEC.md) is.
