# {{COMPANY}} {{DECK_AUDIENCE}} deck outline

Version {{OUTLINE_VERSION | 1.0}} · Updated {{OUTLINE_DATE}} · Owner {{OWNER_NAME}}

> Build agent: fill this with the owner in Phase 4 of the kickoff, from the brief's Block A, E, and F answers and any previous deck Phase 0 found (copy its slide order and look, never its numbers). Ask the question in each _Ask_ line, then delete the _Ask_ lines and this note. Expand `{{#if}}` blocks. Keep the finished outline under 40,000 characters and free of emoji. The first line after the title must stay `Version X.Y …`, and the last line must stay exactly `END OF DECK OUTLINE`: `RP · Stakeholder Deck` checks both on every run. Store it where the brief says, put its URL on the Stakeholder Deck settings row, and raise the version on every change.

What every {{DECK_AUDIENCE}} update contains, where each number comes from, and what it must never say. Read in full by `RP · Stakeholder Deck` on every run. If this outline and the brief disagree, the brief wins and this outline is corrected.

## 1. Audience and purpose

- Audience: {{DECK_AUDIENCE}}. _Ask: who exactly reads it, and what do they use it for (advice, oversight, a funding decision)?_
- The meeting: {{MEETING_RULE_TEXT}}. The deck arrives {{LEAD_WORKING_DAYS | 1}} working day(s) before.
- Tone: factual and plain. Numbers first, then one line of context. No adjectives a number has not earned. No forecasts unless the company-state document states them, labelled as forecasts.
- Confidentiality line, on every slide footer: `{{CONFIDENTIALITY_LINE | Confidential — prepared for <audience>}}`.

## 2. Period and labels

- The deck reports {{REPORT_PERIOD_TEXT | the month the meeting falls in, to date}}.
- Period label: the snapshot's `period.label`, plus `(through <Mon D>)` when `period.is_partial` is true, or `(full month)` when it is false.
- Every date printed is in {{TIME_ZONE}}. The footer carries "Generated <Mon D, YYYY>" in that zone.
- If the snapshot's `period.time_zone` is not {{TIME_ZONE}}, the appendix says so in one line.

## 3. Number rules

1. Every number comes from `payload.json` and nowhere else. `payload.json` is built from the metrics snapshot, the CRM counts, the company-state document, and the sources named in Metric Definitions. The generator never computes a number the payload does not hold, except a rate whose numerator and denominator are both in the payload.
2. Product, transaction, usage, and revenue numbers come **only** from the snapshot (or its run-time stand-in). If the company-state document states a different figure, the snapshot wins; the routine tells the owner and leaves the document's figure out.
3. `null` prints as **not measured**, in the muted style. It never prints as 0. A whole block that is missing omits its slide (section 4).
4. Zero prints as `0`. {{#if ZERO_REVENUE_LABEL}}Revenue of zero prints as `{{ZERO_REVENUE_LABEL}}`, because the brief confirms it is true.{{/if}}
5. Counts print in full with thousands separators. Money prints in major units with the currency symbol, from minor units. Currencies are never summed together.
6. Labels follow each metric's Wording Rule in Metric Definitions. _Ask: any word that must never describe a metric? Defaults: "issued" is not "active"; "signed up" is not "customer"; "listed" is not "booked"; never say money moved "through" the product unless revenue shows it._
7. No number is copied from a previous deck. Month-on-month change, when shown, comes from two snapshot rows read this run.
8. A metric whose definition changed this period carries a footnote: `Definition changed from <Effective From>: <one line>`.

## 4. Slides

The routine builds the slides in this order and omits a slide when its "Omit when" condition holds. It never pads a slide with zeros to keep it. Slide ids are what the generator keys on; do not rename them.

_Ask: walk the list with the owner. Turn slides on or off, reorder, rename titles. Add a slide only with its data keys and its omit rule._

| #   | Slide id             | Title                                 | Data (payload keys)                                                                                         | Render as                                                                                           | Omit when                                                       |
| --- | -------------------- | ------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| 1   | `title`              | {{DECK_TITLE \| <Audience> Update}}   | `period`                                                                                                    | Brand background, logo for dark backgrounds, title, subtitle, period label                          | never                                                           |
| 2   | `snapshot`           | Where we stand · <period label>       | `render.metrics` where `headline` is true (at most 6)                                                       | Stat cards: number, label, one sub-line (cumulative or context)                                     | never; a missing headline shows **not measured**                |
| 3   | `leadership`         | Leadership                            | `narrative.leadership_additions`                                                                            | One card per new person: name, role, start date, mandate                                            | the key is absent or empty                                      |
| 4   | `leadership_profile` | <Name>                                | one entry of `narrative.leadership_additions`                                                               | Role, mandate, start date, factual highlight bullets                                                | the key is absent; at most 2 profile slides                     |
| 5   | `segments`           | People by {{SEGMENT_NAME \| role}}    | `segments.by_role`                                                                                          | Drawn horizontal bars, cumulative value labelled, `+new` beside it; totals to the right             | the key is absent                                               |
| 6   | `supply`             | {{SUPPLY_TITLE \| Growth and supply}} | `render.metrics` with Block Growth, Revenue, or Marketplace that are not headlines; `marketplace.inventory` | Stat cards                                                                                          | none of those metrics and no inventory                          |
| 7   | `usage`              | {{USAGE_TITLE \| Product usage}}      | `render.metrics` with Block Usage or Community; `usage.measured_since`                                      | Label and value rows; error rate from numerator and denominator; `measured_since` footnote          | no Usage or Community metric is in the deck                     |
| 8   | `gtm`                | Go-to-market                          | `render.metrics` with Block CRM; `crm.pipeline`; `crm.caveat`                                               | Stat cards for period counts; a pipeline strip by Status (cumulative); the CRM Caveat as a footnote | `crm` is absent                                                 |
| 9   | `capital`            | Capital and milestones                | `narrative.funding.wins`, `narrative.funding.milestones`                                                    | Up to 3 cards (amount as stated, name, status verbatim); a dated milestone list (up to 5)           | both lists are absent or empty                                  |
| 10  | `corporate`          | Corporate                             | `narrative.corporate`                                                                                       | Up to 3 stat cards and a short "in motion" list                                                     | the key is absent                                               |
| 11  | `decisions`          | Decisions this period                 | `narrative.decisions`                                                                                       | One line per decision: the resolution, the date, one line of why                                    | the key is absent or empty                                      |
| 12  | `admin_changes`      | Administrative changes                | `narrative.administrative_changes`                                                                          | Category tag, title, date, one detail line (up to 6)                                                | the key is absent or empty                                      |
| 13  | `roundup`            | Notable this period                   | `narrative.notable`                                                                                         | Numbered list, title and one line each (3 to 5)                                                     | the key is absent or empty                                      |
| 14  | `risks`              | Risks and open items                  | `narrative.risks`                                                                                           | One line each: risk, owner, next step                                                               | {{RISKS_OMIT \| always, unless the brief turned this slide on}} |
| 15  | `closing`            | Questions and discussion              | `period`                                                                                                    | Brand background, logo, meeting name                                                                | never                                                           |
| 16  | `appendix`           | Sources and definitions               | `render.metrics` (definition, source, as of); `unmeasured`; `render.omitted`                                | Small table: metric, definition, source, as of. Then every not-measured key and every omitted slide | {{APPENDIX_OMIT \| never}}                                      |

## 5. Narrative rules

The narrative slides (3, 4, 9 to 14) come from the company-state document, the decisions log, and the Highlights table. They add facts, never product numbers.

- **Company-state document:** {{COMPANY_STATE_DOC | none}}. The newest dated version only; archived copies are ignored. _Ask: which section feeds which slide?_

  | Slide         | Section of the company-state document |
  | ------------- | ------------------------------------- |
  | leadership    | {{SECTION_LEADERSHIP}}                |
  | capital       | {{SECTION_FUNDING}}                   |
  | corporate     | {{SECTION_CORPORATE}}                 |
  | admin_changes | {{SECTION_GOVERNANCE}}                |
  | roundup       | {{SECTION_NOTABLE}}                   |

- **Internal sections never go in a deck:** {{COMPANY_STATE_INTERNAL_SECTIONS | none named}}. _Ask: which sections are internal (unreconciled items, sensitive negotiations, anything said in confidence)?_ They stay out even when they are the most interesting thing that happened.
- **Decisions log:** {{DECISIONS_LOG | none}}. Only decisions with status {{DECIDED_STATUSES | Locked}} dated in the period. The resolution and one line of why. The log supplies reasons, never numbers.
- **Highlights:** rows with Include on and Period equal to the deck's period.
- **People:** a person appears on the leadership slide only while their start date is within {{NEW_PERSON_WINDOW_DAYS | 60}} days of the period end. Highlight bullets are facts the document states (signed date, start date, scope of role). Never an invented background, never a guessed title.
- **Funding:** the amount, name, and status exactly as the document states them. A programme applied for is not a programme won.
- **Optional keys.** Every narrative key is optional. A key the sources do not support is left out, and its slide is omitted. Nothing is fabricated to fill a slide.

## 6. Never in the deck

- Anything in an internal section named in section 5.
- Personal data beyond a new team member's name, role, and start date.
- Customer or counterparty names, unless {{NAMED_CUSTOMERS_RULE | the company-state document names them as public}}.
- A number with no source in `provenance` or in this outline's rules.
- A claim that money moved through the product when revenue does not show it. Credits, waivers, and pilots are described precisely or not at all.
- {{NEVER_IN_DECK_EXTRA}}

## 7. Look

_Ask: read these from the brand pack or design tokens if Phase 0 found them. The snapshot's `brand.tokens` override this table when present._

| Role                    | Value                                                                                                               |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Primary                 | `{{COLOUR_PRIMARY}}`                                                                                                |
| Primary dark            | `{{COLOUR_PRIMARY_DARK}}`                                                                                           |
| Secondary               | `{{COLOUR_SECONDARY}}`                                                                                              |
| Accent                  | `{{COLOUR_ACCENT}}`                                                                                                 |
| Background              | `{{COLOUR_BACKGROUND \| #F5F5F5}}`                                                                                  |
| Surface                 | `{{COLOUR_SURFACE \| #FFFFFF}}`                                                                                     |
| Text / muted            | `{{COLOUR_TEXT \| #262626}}` / `{{COLOUR_MUTED \| #737373}}`                                                        |
| Danger                  | `{{COLOUR_DANGER}}` (error counts only)                                                                             |
| Heading font            | {{FONT_HEADING}}                                                                                                    |
| Body font               | {{FONT_BODY}}                                                                                                       |
| Logo, light backgrounds | {{LOGO_LIGHT_LOCATION}}                                                                                             |
| Logo, dark backgrounds  | {{LOGO_DARK_LOCATION}} {{#if LOGO_RECOLOUR}}(recolour to white, keeping transparency: the file is not white){{/if}} |

- Layout 16:9. Title and closing slides on the primary colour; content slides on the background colour.
- Charts are drawn bars with a text label and a value on every bar, or images rendered by a plotting library with the same labels. Native chart objects are avoided: some viewers drop their labels.
- At most 6 stat cards on a slide. Nothing overlaps the footer.
- If a logo cannot be read, the title and closing slides use a text wordmark in the heading font.

## 8. File

- Format: {{DECK_FORMAT | pptx}}.
- File name: `{{DECK_FILE_NAME | <Company> <Audience> Update - <Period label>}}.<ext>`. A rebuild of a delivered period adds ` v2`, ` v3`.
- Folder: {{DECK_FOLDER}}. Dry runs and test periods go to its `Dry runs` subfolder.
- The `payload.json` the deck was built from is saved next to it as `payload - <Period label>.json`.

## Change log

YYYY-MM-DD · v1.0 First version from the kickoff ({{OWNER_NAME}})

END OF DECK OUTLINE
