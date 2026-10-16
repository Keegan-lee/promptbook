# Blog kit

Two halves. **Build** the publication once. **Run** the writing engine every week.

Hand this folder and `prompts/_shared/` to your agent. The kit is company-agnostic: the interview fills in your business, your market, and your tools.

## Files

| File                           | Half  | Job                                                                                                                                             |
| ------------------------------ | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `SPEC.md`                      | Build | The publication contract: interview, Site brief, CTA catalog, CMS schema, routes, live gate, SEO, AEO, GEO, `llms.txt`, `robots.txt`, sitemap.  |
| `KICKOFF.md`                   | Run   | The one prompt you hand your agent to set up the writing engine.                                                                                |
| `KEYWORD_TOOL.md`              | Run   | The keyword-tool contract every routine uses: endpoints, thresholds, budgets, mappings for Ubersuggest and four alternatives.                   |
| `SCHEMA.md`                    | Run   | The Airtable backlog: Blog Backlog, Keyword Ledger, Blog Settings, Run Log, views, interface, automations. Plus the existing-task-table option. |
| `TICKET_TEMPLATE.md`           | Run   | The body of every blog ticket.                                                                                                                  |
| `routines/content-gap-scan.md` | Run   | `BL · Content Gap Scan`: weekly keyword research that writes tickets.                                                                           |
| `routines/post-writer.md`      | Run   | `BL · Post Writer`: turns an approved ticket into a CMS draft a person reviews and publishes.                                                   |
| `routines/refresh-review.md`   | Run   | `BL · Refresh Review`: checks live posts that are due for review and proposes refresh or correction tickets.                                    |
| `MANUAL.md`                    | Run   | The steps only a person can click: views, interface, automations, connectors, CMS token scopes, and the publish step.                           |
| `OPERATORS.md`                 | Run   | The operator's manual template. The agent fills it from the brief and hands it to the team.                                                     |

## SPEC.md versus the writing kit

|                 | `SPEC.md`                                          | Writing kit (`KICKOFF.md` and the rest)                         |
| --------------- | -------------------------------------------------- | --------------------------------------------------------------- |
| Builds          | The CMS schema, routes, rendering, structured data | The backlog, the keyword research, the scheduled jobs           |
| Runs            | Once, then on schema changes                       | Every week                                                      |
| Output          | A publication that can hold posts and gate them    | Tickets, CMS drafts, refresh proposals                          |
| Who uses it     | A developer agent in the codebase                  | An operations agent with Airtable, CMS, and keyword-tool access |
| Brief it writes | The **Site brief** (with the CTA catalog)          | The **Blog writing brief**, which reads the Site brief          |

The writing kit never re-asks what the Site brief already answers: audiences, CTA catalog, categories, embeds, numbers that must never be typed, reviewers, keyword tool and market. It reads them and confirms them.

## Order

1. **Run `SPEC.md`** if the publication does not exist yet. It ends with a filled Site brief and a CMS that has drafts and a `live` gate.
   - If a blog already exists and you are not rebuilding it, the kickoff records the existing post model instead (Phase 0).
2. **Run `KICKOFF.md`.** Interview, brief, backlog, keyword tool, routines, verification, hand back.
3. **Operate** from `OPERATORS.md`: approve topics, review drafts, publish, answer blocked tickets.

## What to hand an agent

- This `blog/` folder and `prompts/_shared/` (`INTERVIEW.md`, `AIRTABLE.md`, `ROUTINES.md`).
- The sentence: **"Read `prompts/blog/KICKOFF.md` and run the kickoff."**
- Access, or a person who can grant it during the interview:
  - Airtable (the base the backlog lives in);
  - the keyword tool (Ubersuggest by default);
  - the CMS, with a token that can create drafts;
  - the agent scheduler (cloud routines, or desktop scheduled tasks);
  - chat, for digests and "draft ready" messages.
- Never paste a key or password into the chat. The agent asks you to add each one in the connector or secret-store UI.

## What the engine never does

- Publish, unpublish, delete, or switch a post live. A person does that. (If the brief allows it, the writer creates supporting documents such as glossary terms published with the live flag off, so a draft can reference them; switching them live stays a person's.)
- Set editorial flags (featured, pinned) or hand-write an embed, a share token, or an iframe.
- Change a slug or silently retarget a live URL.
- Type a number it did not read from a tool, a source it opened, or your data. Keyword numbers carry the tool and the date they were pulled.
- Type a number the Site brief says must never be typed (live counts that drift).
- Call the keyword tool's expensive endpoints (article generation, metric recalculation).

## Related kits

- **Content OS** (optional): turns a newly live post into social posts. Its nightly CMS inventory sync picks up every live post; the writer links each ticket to that Content row. Its own article-to-CMS routine stays off while this kit drafts posts.
- **Sales Pipeline** (optional): its writing module may link a live post in an email when it helps that reader ("content matching"). It reads live posts only.
