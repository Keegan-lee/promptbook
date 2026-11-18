# Ticket standard (template)

Version 1.0 · Owner: {{OWNER_NAME}}

> Build agent: fill every `{{…}}` from the brief, keep the `{{#if CODE}}` blocks and the **Code only** rows only if the team ships code from the tracker, save the result where the brief lives, and record its location and version in the brief. This is a **binding document**: every agent that writes or works a ticket reads it, and the routines read it on every run. Keep it free of emoji and under 40,000 characters, and keep the last line exactly as it is.

This is how {{COMPANY}} writes tickets in the {{BASE_NAME}} base, and how anyone, person or agent, works one. A ticket is good when someone who has never seen the conversation that produced it can pick it up and finish it.

## 1. What every ticket carries

| Field        | On create                                                                                                                                                                                                                                         |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Title        | At most 90 characters. Shapes below.                                                                                                                                                                                                              |
| Status       | `Raw`, unless a person said otherwise. A brief that already passes the readiness checklist (section 4) may go straight to `Backlog`. Never `Todo` or later.                                                                                       |
| Project      | Exactly one existing project. **Never create a project.** If none fits, ask, or use {{DEFAULT_PROJECT}} and say so in PM Notes.                                                                                                                   |
| Assignee     | {{DEFAULT_ASSIGNEE \| empty: the prioritiser assigns}}. When it is an agent persona, set Engine too.                                                                                                                                              |
| Priority     | Critical, High, Medium, or Low, by the meanings in section 6.                                                                                                                                                                                     |
| Type         | One or more existing options. What kind of work, not which area.                                                                                                                                                                                  |
| Tags         | The epic's tag, if it belongs to one, and area tags that fit. Existing options only. A new tag is a person's decision.                                                                                                                            |
| Effort       | {{EFFORT_SCALE \| Fibonacci only: 1, 2, 3, 5, 8, 13}}. Estimate for the whole ticket, review included. 13 or more: split it.                                                                                                                      |
| Complexity   | trivial, standard, or complex.                                                                                                                                                                                                                    |
| Source       | manual, form, agent, routine, error-tracker, or import.                                                                                                                                                                                           |
| Engine       | **Code only.** Who or what should execute it: {{ENGINES}}. {{ENGINE_RULE \| The most capable agent by default; a small local model only for mechanical one-file changes; human for anything that needs a login, a payment, or a judgement call.}} |
| Dependencies | Every ticket that must be closed before this one can start. Link by record id.                                                                                                                                                                    |

## 2. Titles

| Kind       | Shape                                                 | Example                                              |
| ---------- | ----------------------------------------------------- | ---------------------------------------------------- |
| Work       | `<Area or epic>: <what will be true when it is done>` | `Checkout: refunds show the original payment method` |
| Bug        | `Bug: <where> — <what happens>`                       | `Bug: POST /orders — 500 when the cart is empty`     |
| Epic       | `Epic: <outcome>`                                     | `Epic: self-serve onboarding for new sellers`        |
| Discussion | `Decide: <question>`                                  | `Decide: do trial accounts get exports?`             |
| Regression | `<original title> (regression of {{ID_PREFIX}}-<n>)`  |                                                      |

## 3. The description is a prompt

Write the Description so the assignee can execute it cold: no other context, no "as discussed". Use the shape for the ticket's kind. Name real files, documents, and links; never "the usual place".

{{#if CODE}}

### Code, feature, and build tickets

```
**Epic:** <Epic title> ({{ID_PREFIX}}-<n>). Key **<KEY>-<nn>**. <one line of why>
(leave this line out when the ticket has no epic)

<Two to four sentences: what is true today, and what must be true after.>

## Read first
- `<path/to/file>`: <why it matters>. Imitate the pattern in `<path>`.
- `<doc or spec path>` §<section>: <what it settles>.

## Task
1. <A specific deliverable: a table and its columns, an endpoint and its method, a component, a job.>
2. …

## Constraints
{{STANDING_CONSTRAINTS}}
- <Anything specific to this ticket.>

## Acceptance
- <A check someone can run and see pass or fail.>
- <Tests: which, where, green.>
```

### Bug tickets

```
**Symptom:** <exact error text, trimmed>. Where: <route, job, screen>. <events> events, <users> users, first seen <date>, last seen <date>, environment <env>. <link>

**Evidence:**
<the top stack frames or log lines, verbatim, in a code block, with anything that looks like an email address, token, or key removed>

## Read first
- `<path>` …

## Task
1. Reproduce it, or explain the root cause from the code.
2. Fix it.
3. Add or extend a test that fails before the fix and passes after it.
4. Open a PR into `{{PR_BASE_BRANCH | the default branch}}`.

## Constraints
{{STANDING_CONSTRAINTS}}

## Acceptance
- The error stops in the error tracker within a day of the deploy.
- The new test fails before the fix and passes after it.
```

{{/if}}

### Non-code tasks

```
## Goal
<What will be true when it is done, and for whom.>

## Context
- <The documents, records, people, and links the task needs. Real links.>

## Steps
1. <Concrete steps, in order.>

## Done when
- <Something a person can check: a filed document, a sent draft, a booked date.>
```

### Discussions and decisions

```
## Question
<One question.>

## Options
1. <Option>: <cost, risk, what it unlocks>.
2. …

## Who decides, and by when
<Name> · <date or "before <{{ID_PREFIX}}-n> starts">
```

The decision is recorded in PM Notes, and the ticket closes as Done. Tickets the decision unblocks name it in their PM Notes.

## 4. Readiness checklist (Raw or Needs Grooming → Backlog)

A ticket is ready when all of these hold:

- The Title follows section 2.
- The Description has every section of its shape, and none is a placeholder: no `<…>`, no template braces, no "TBD".
- The acceptance (or "Done when") lines are checks, not wishes.
- Project, Priority, Type, Effort, and Complexity are set.
- Every ticket it waits for is in Dependencies.
- Every business fact it needs is written down. A missing one is a question, not a guess: move the ticket to Needs Grooming and make the question the last PM Notes line.

## 5. PM Notes

The decision log. Later decisions live here, and **PM Notes wins** when it and the Description disagree.

- Append-only, oldest first, so the log reads in order. One line per entry: `[YYYY-MM-DD] <who>: <what>`. Other kits' routines may write the shared notes shape, `YYYY-MM-DD · <routine>: <what>`; both count as dated lines.
- Read the field, then write the old text plus the new line. The API replaces the whole field on update: a write that sends only the new line erases the log.
- When a decision changes the brief, rewrite the Description too, and log a line saying so (`[2026-03-04] Sam: scope cut to card payments; Description rewritten`).
- A routine that created the ticket writes one provenance line: `[YYYY-MM-DD] <routine>: created from <source> · Key: <dedupe key>`.
- Blocked: the last line names the reason and who unblocks it. Needs Grooming: the last line is the question. QA: the last line is the evidence. Won't Do: the last line is why.
- Never paste a secret, a password, a customer's personal data, or a whole email into PM Notes.

## 6. Priority

> Build agent: replace this table with the brief's meanings (KICKOFF Block C) if they differ, and delete this note.

| Priority | Means                                                                                                             |
| -------- | ----------------------------------------------------------------------------------------------------------------- |
| Critical | Money moves wrongly, data is lost or exposed, a legal duty is missed, or a core journey is down. Drop other work. |
| High     | A customer is blocked or a launch is waiting on it. This cycle.                                                   |
| Medium   | Worth doing soon; a workaround exists.                                                                            |
| Low      | Nice to have, cosmetic, internal-only.                                                                            |

## 7. Epics

An epic is a ticket (Type `Epic`, Title `Epic: <outcome>`) whose Dependencies are its children.

1. **Write the epic first.** Description sections: `## Goal`, `## Rules` (decisions every child must follow), `## Phases` (children keys per phase), `## Rollout gates` (steps only a person may take, such as a production migration, a switch, a payment setting), `## Later (filed, do not start)`, `## Out of scope`.
2. **Pick a key** for the epic, two to four capitals (`ONB`). Children are `ONB-01`, `ONB-02`, …; the key is the first line of each child's Description, and every child carries the epic's tag. Create the tag option at build time; routines never mint it.
3. **Create every child** (pass 1). Capture each new record id.
4. **Link** (pass 2): each child's Dependencies are its blockers; the epic's Dependencies are all of its children. A link needs both records to exist, which is why this is a second pass.
5. **Write the ticket map** as the epic's first PM Notes line: `Ticket map: ONB-01 {{ID_PREFIX}}-<n> <record id> · ONB-02 …`. Name the day-one roots (children with no blockers).
6. The epic stays `Raw` or `Backlog` while its children are worked, moves to `In Progress` when the first child does, and goes to QA when `Open Dependencies` reaches 0.

## 8. Working a ticket

Every person and every agent that picks up a ticket does this.

1. **Read** the Description, then PM Notes (PM Notes wins), then the Dependencies. Do not start until every dependency is closed, unless a person says so in PM Notes.
2. **Claim it:** set Status to `In Progress` before you start, and Assignee if it is empty.
3. **Missing fact:** stop and ask. Move it to `Needs Grooming` with the question as the last PM Notes line. Never invent a business fact to keep going.
4. **Blocked by something outside the ticket:** `Blocked`, with the reason and who unblocks it.
5. **Scope changed while you worked:** log it in PM Notes before you hand back. A silent change is the one the reviewer misses.
6. **Finished:** append the evidence to PM Notes: {{#if CODE}}the PR link (also in PR Link), the branch, {{/if}}what you ran and what it showed, the ids of anything you created. Then set `QA`. A ticket in QA without evidence goes back.
7. **Never** set `Todo`, `Done`, or `Won't Do`. Never delete a ticket. Never move a ticket backward out of QA or Done; reopen work as a new ticket that names the old one.
8. One piece of work that spans several tickets: claim them all, and move them all to QA together, each with its own evidence line.

## 9. Rules for routines that write tickets

- Before creating, look the ticket up by **Dedupe Key**, exact match. Found and open: append one PM Notes line at most per run, never a second ticket. Found and Done: a regression ticket that names the old one. Not found: create.
- Write the Dedupe Key, the Source, and the provenance line on every ticket you create.
- Read each select field's live options before writing. Write only options that exist. Never pass `typecast: true`.
- Never create a project, a tag, a person, or a select option.
- Link by record id, never by name.
- Respect the run cap in Tracker Settings. Name the rest in the digest as "not ticketed this run".
- Status on create: `Raw`. Never higher.

END OF TICKET STANDARD
