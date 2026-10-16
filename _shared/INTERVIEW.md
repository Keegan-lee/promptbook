# Interview protocol (every kit)

Every kit in this directory is company-agnostic. The **brief** is what makes it yours. This file says how the agent running a kit interviews the person, records the answers, and turns them into a build. Each kit's `KICKOFF.md` adds its own question blocks on top of the shared block below.

## Rules

1. **Interview before you build.** No base, table, routine, document, or workflow is created until the brief is written down and the person has said "go" to it.
2. **Two kinds of answer.**
   - **Business facts** (what you sell, to whom, what you may claim, who approves, which addresses you may email). Never invent these. Ask. If the person doesn't know, record `UNKNOWN` and turn off whatever depends on it.
   - **Mechanics** (caps, gaps, thresholds, batch sizes, schedules). Every kit ships a proven default. Offer it in brackets and accept "default".
3. **Discover before you ask.** Before the first question, list your connected tools (MCP servers, connectors, CLIs) and search for things that already exist: an Airtable base with a similar name, a brand guide, a sales playbook, a CMS, a calendar booking link. Ask "I found X. Use it?" instead of asking from scratch.
4. **One block per message.** Ask 3–6 numbered questions at a time. Show the default after each question in brackets. The person may answer `default`, `skip` (feature off), or `later` (recorded as an open item; the build continues without it).
5. **Adapt the depth.** A solo founder and a twelve-person team do not need the same build. Ask the scale question early (Block 0, question 6) and skip blocks that the answer makes irrelevant. Say which blocks you skipped and why.
6. **Use their words.** Record answers close to verbatim. Vocabulary from the interview (audience names, stages, channel names) becomes select options and field names.
7. **Never ask for a secret in chat.** API keys, passwords, OAuth tokens, and webhook secrets go into the platform's connector UI, secret store, or environment settings. Ask the person to add them there and tell you when it is done. Briefs never record bank or card numbers, tax ids, or government ids: refer to each account by a nickname ("operating account", "company card").
8. **Write the brief to a durable place.** Ask where: a file in their repository, a doc in their drive, or a `Kit Settings` record in Airtable. A brief that only lives in chat is lost when the session ends. Keep every kit's brief in the same place, next to the Company brief (Block 0).
9. **Read it back.** Show the finished brief, list every `UNKNOWN` and `later`, list every default you applied, and get an explicit "go".
10. **The brief is the source of truth.** Later changes go into the brief first, then into the system. If the brief and the live system disagree, ask which one is right before you touch either.

## Block 0 — Company (shared by every kit)

Block 0 is answered **once per company**, not once per kit. The first kit writes the answers to a standalone **Company brief** (`# Company brief — <Company>`, same layout rules as below). Every later kit reads it, shows it back, and asks only "anything changed?". A change goes into the Company brief first, with a change-log line, and every kit brief points at it rather than copying it. The Company brief also records the shared pointers other kits read: the company-state document and its pointer record (the SPINE kit), the CRM base, and the chat destinations.

1. The company name as it appears in running text, and a one-sentence description a stranger could repeat.
2. The public website origin (`https://…`).
3. The home time zone as an IANA name (for example `America/New_York`), the working days, and the public-holiday calendar to skip. [Mon–Fri, national holidays of the home country]
4. The language and spelling variant (for example `en-US`, `en-GB`, `en-CA`, `fr-CA`).
5. The **owner**: the one person who approves, holds the kill switches, and is the default sender. Name, role, email, and every alias they send from.
6. The team: everyone else who appears in the work, with role, email, and aliases. Say whether there is one person, a small team (2–10), or more. Do people share a single login to any tool? [assume yes: every automated write is stamped with who and why]
7. Internal domains and personal addresses that count as "us" (so they are never treated as prospects or counterparties).
8. Where routines report: the chat tool and the channel or DM for digests and errors. [owner's DM]
9. Where the brief should live.

## Capability check

Each kit lists the capabilities it needs. Build a table like this one in the brief, filled from what you discovered and what the person told you:

| Capability                 | Needed by          | Their tool | Connected? | If missing                           |
| -------------------------- | ------------------ | ---------- | ---------- | ------------------------------------ |
| Operational database       | every kit          | Airtable   | yes / no   | stop: every kit is built on Airtable |
| Scheduler for agent jobs   | every routine      |            |            | run by hand until one exists         |
| Workflow engine            | deterministic jobs |            |            | an agent routine does the job        |
| Chat notifications         | digests, errors    |            |            | email the owner instead              |
| (kit-specific rows follow) |                    |            |            |                                      |

A missing capability is never a reason to fake one. Either pick the stated fallback or mark the feature `off` in the brief.

## Placeholders

Kit files use `{{UPPER_SNAKE_CASE}}` placeholders, for example `{{COMPANY}}`, `{{OWNER_NAME}}`, `{{TIME_ZONE}}`, `{{ID_PREFIX}}`. Fill every placeholder from the brief or from the ID map the build produces (`_shared/AIRTABLE.md`).

- A routine or document with an unfilled `{{…}}` is not ready. Search for `{{` before you create anything.
- `{{#each MOTIONS}} … {{/each}}` and `{{#if FEATURE}} … {{/if}}` blocks mean what they look like: repeat per item, or keep only when the brief turned the feature on. Remove the markers when you fill them.
- Where a kit gives a **default**, it is written inline as `{{NAME | default}}`.

## Brief layout

Every kit's brief uses this shape, so briefs from different kits sit side by side:

```
# <Kit name> brief — <Company>
Version: 1.0 · Written: YYYY-MM-DD · Owner: <name>

## Company            (Block 0, shared)
## Capabilities       (the table above)
## <Kit blocks>       (one section per interview block, answers close to verbatim)
## Defaults applied   (every mechanic left at its default)
## Open items         (every UNKNOWN and "later", with what is switched off until it is answered)
## ID map             (filled after the build: base, tables, fields, select options, record ids)
## Change log         (dated, one line each)
```
