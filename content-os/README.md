# Content OS kit

A portable marketing pipeline run from Airtable: ideas become content, content becomes one post per channel, draft designs come from your brand templates, a person approves every post and every design, and jobs schedule, send, confirm, and measure each one. It is company-agnostic. The interview in `KICKOFF.md` makes it yours.

**To use it:** give your agent this folder plus `../_shared/`, and say "Run the Content OS kickoff."

| File                               | What it is                                                                                                                      |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `KICKOFF.md`                       | The prompt. Discover → interview → brief → base → brand pack → workflows → routines → go-live ladder.                           |
| `SCHEMA.md`                        | The Airtable field contract: content, posts, channel rules, assets, the gate, scores, automations, views, interfaces.           |
| `BRAND_PACK_TEMPLATE.md`           | The brand pack every writing and design step reads on every run, and the design tool's brand kit and template setup.            |
| `routines/copy-writer.md`          | `CP · Copy Writer`: one draft post per channel from Briefed content; revisions sent back by reviewers.                          |
| `routines/creative-drafts.md`      | `CP · Creative Drafts`: approved copy becomes a draft design from the channel's brand template; a person finishes and approves. |
| `routines/idea-miner.md`           | `CP · Meeting Idea Miner`: opt-in meetings become redacted, generic ideas.                                                      |
| `routines/article-to-cms-draft.md` | `CP · Article → CMS Draft`: an approved article becomes a CMS draft. Only without the blog kit.                                 |
| `workflows/README.md`              | Shared rules and production gotchas for the deterministic jobs.                                                                 |
| `workflows/*.md`                   | Error handler, CMS inventory sync, scheduler, distribute (or its agent-routine variant), confirm posts, analytics, media links. |
| `MANUAL.md`                        | What a person must click: views, interfaces, stamps, connections, brand kit, templates, designs.                                |
| `OPERATORS.md`                     | The operator's manual template the agent fills in at the end.                                                                   |

## What it never does

- Post anything a person has not approved with their own click, on the post **and** on every design or file the post carries.
- Post through any job but Distribute, or schedule inside the distributor where an un-approval cannot reach it.
- Guess a distributor profile, a page id, a live URL, or a number. Live counts are never typed at all.
- Publish to a CMS, or change a brand template.
- Read a meeting nobody opted in, or let a name, an amount, or a quote from one reach Airtable.

## Order

Core first, then one module at a time, each after the last runs clean for two weeks (`KICKOFF.md`, "Start small"):

1. **Core:** Content, Platform Posts, Platform Rules, Ideas, Pipeline Errors, Settings, the stamps, and `CP · Copy Writer`. Posting is manual.
2. **Distribution:** Assets, media links, Scheduler, Distribute, Confirm Posts, Error Handler. A workflow engine by default; one agent routine with the distributor's connector when there is no engine. Same gate either way.
3. **Creative:** the brand kit, brand templates in the Templates table, the asset stamp, and `CP · Creative Drafts`.
4. **Analytics:** Analytics Pull, Engagement Score, the weekly digest.
5. **Meetings, long-form, CMS, articles:** as the brief turns them on.

## Related kits

- **Blog** (`../blog/`): drafts posts into the CMS. With it installed, this kit's article routine stays off, and the CMS inventory sync turns each newly live post into Briefed content for social.
- **Sales Pipeline** (`../sales-pipeline/`): may archive meeting transcripts that the Idea Miner reads, through the opt-in log only. The Idea Miner never opens the CRM.
