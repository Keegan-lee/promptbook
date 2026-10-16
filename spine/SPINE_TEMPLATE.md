# SPINE template

> Build agent: this is the skeleton of the SPINE document. In the kickoff's Phase 5 you compose version 1 with the owner from the brief and a first read of every source. Everything below the line becomes the document. Fill every `{{…}}` from the brief, replace every `<…>` with real content or delete the line, and keep the fixed parts exactly as written: the header lines, §0, §1, the Deadlines and Open items tables, the corrections log, the footer, and the `END OF SPINE` line. The other sections come from the brief's section list (`{{SPINE_SECTIONS}}`); rename, merge, or drop them there, not here.
>
> **What belongs in the SPINE: what moves.** Current numbers, live deadlines, open items, who is who now, the few rules that bind every decision. **What does not: what has solidified.** History, background, legal analysis, full rosters and lists, the reasoning behind a decision, anything that explains rather than states. Solidified context goes into its own document, is registered in the Data Room as an Article (State Role `SPINE Extract`), and is cited here in one line. The SPINE moves; settled context does not move with it.
>
> **Writing rules.**
>
> - Every number is typed once, in the numbers or traction section. Other sections cite the section (`§5`), never retype the figure. Two copies of a number drift.
> - Every document is cited by its Data Room Article id. Every decision by its Decision id. A signed document may also carry its own reference (a filing number, an e-signature envelope id).
> - Every figure carries its source and an as-at date. A figure taken from a snapshot rather than a live query names the snapshot's date.
> - An inferred value is labelled as inferred. A system of record that has not caught up with the calendar is reported as it stands ("date passed, the system still reads confirmed"), never advanced by hand.
> - When the raw count in a system and the true count differ (duplicate rows, zero-value rows, test records), give the true count and say why in one clause.
> - One line per item. Tables for dates and metrics. No emoji. Plain Markdown that survives export.
> - Keep the whole document under `{{WORD_CEILING | 2500}}` words and 40,000 characters. A rough budget: header 150, §0 40, §1 up to 200, at a glance 180, phase 100, deadlines 300, numbers 300, traction 400, where things live 100, fronts 100, open items 350, dormant 40, corrections log 220. It is a guide, not a rule; the ceiling is the rule.
>
> **Standing facts** (§1) are facts the owner has settled so no agent asks again. Only the owner adds, edits, or removes one. The sync carries the section forward word for word. If the brief keeps standing facts on the settings row instead (`{{STANDING_FACTS_HOME | SPINE}}`), §1 holds a one-line pointer to that field.

---

# SPINE — {{COMPANY}}

Version: <YYYY-MM-DD> · Last verified: <YYYY-MM-DD> · Maintainer: {{OWNER_NAME}} · Supersedes: <prior file name, or "none (version 1)"> · Words: <n> of {{WORD_CEILING | 2500}}

**What this is.** The canonical current state of {{SPINE_SCOPE}}. It holds what moves: current numbers, live deadlines, open items, who is who now. Settled context lives in its own documents, registered in the Data Room and cited here in one line. Read it first. Do not pin this file's id: resolve the pointer record each time.

**Authority order.** {{AUTHORITY_ORDER | signed agreement → this file → current system state → unsigned draft → memory}}. Against a signed agreement, the agreement wins and this file is wrong. A conflicting unsigned draft is stale. {{#each SYSTEMS_OF_RECORD}}For {{FACT_CLASS}}, {{SYSTEM}} is the system of record and outranks this file. {{/each}}Never call a transaction complete, or a payment made, unless its system of record shows it.

**Citing.** Numbers are typed once, in §5 and §6; everything else cites them. Documents are cited by Data Room Article id; decisions by Decision id. An open conflict keeps its old wording and carries `[UNRECONCILED <date> — see Observation <id>]` until the owner settles it.

## 0. How agents work here

Read before acting: {{AGENT_PROTOCOL_NAME | Agent Operating Protocol}} — Article <id>.{{#each WORKSTREAM_INSTRUCTIONS}} {{WORKSTREAM}} work also runs under <document name> — Article <id>.{{/each}}

## 1. Standing facts

Settled by {{OWNER_NAME}}. No agent re-raises these. Only {{OWNER_NAME}} adds, edits, or removes a line.

- <YYYY-MM-DD> · <the fact, in one sentence> · <evidence: Article or Decision id, or "owner's confirmation">

## 2. Company at a glance

Entity, registry, governing law, year end: Article <id>.

Live rules: <the few rules that bind every decision, one clause each, each with its Decision id and status>.

People: <who is who now: roles, who decides what, who has left and what still binds them>.

## 3. Current phase

<One paragraph: what the company is doing now and what "done" looks like for this phase.>

## 4. Deadlines

| Date         | What                       | Consequence                                     |
| ------------ | -------------------------- | ----------------------------------------------- |
| <YYYY-MM-DD> | <what is due, and to whom> | <what happens if it slips; who holds the right> |

Sorted by date. A passed internal deadline gets a new date and a one-line consequence. A passed deadline owed to a third party (a funder, a regulator, a counterparty) stays until {{OWNER_NAME}} settles it.

## 5. Canonical numbers

<Financing, ownership, obligations, runway, headcount: each figure once, with its source and as-at date. Long tables (a full ownership table, a debt schedule) live in an extract Article; keep the totals and the pointer here.>

## 6. Business and traction

<Each product line in one or two lines: what it is, its status, the Article that holds its rules.>

Traction as at <YYYY-MM-DD> — {{TRACTION_SYSTEM}}, re-queried this cycle.

| Metric                               | Value                                                 |
| ------------------------------------ | ----------------------------------------------------- | --------- |
| {{#each TRACTION_METRICS}}{{METRIC}} | <value, with the true-versus-raw note if they differ> | {{/each}} |

**Say it precisely.** <The one sentence anyone may repeat outside the company, using the figures in the table.> {{#each PRECISION_RULES}}{{RULE}} {{/each}}

## 7. Where things live

Systems, owners, and join keys: Article <id>. <Other pointers, one line each: where the signed set lives, where the backlog lives.>

## 8. Live fronts

<At most {{MAX_FRONTS | 4}} fronts, one line each, in priority order.>

<The rule that breaks ties, for example: "front 1 outranks the rest, except where a dated obligation in another bites first".>

## 9. Open items

| Item                        | Detail                                                                |
| --------------------------- | --------------------------------------------------------------------- |
| <short name of the problem> | <what is open, what settles it, who acts, the Article or Decision id> |

A resolved item is deleted, never annotated.

## 10. Dormant and deferred

Article <id>. <Anything that looks dormant but is not, one line.>

## 11. Corrections log — changes from <prior file name>

<Only the changes from the prior version to this one, grouped by section. Older logs are dropped.>

EXTRACTED: <what moved out, to which Article id, and the pointer line left behind>.

Not gathered: <each source that was unavailable this run, and for how many consecutive runs>.

{{SPINE_FOOTER | Internal record. Not legal, tax, or financial advice.}}

END OF SPINE
