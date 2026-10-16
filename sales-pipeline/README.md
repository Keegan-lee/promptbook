# Sales Pipeline kit

A portable sales system: a CRM that keeps itself truthful, a radar that makes sure nobody is left waiting on a reply, and evidence-backed outreach in the owner's voice, all steered from one control panel. It is company-agnostic. The interview in `KICKOFF.md` makes it yours.

**To use it:** give your agent this folder plus `../_shared/`, and say "Run the Sales Pipeline kickoff."

| File                                | What it is                                                                                     |
| ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| `KICKOFF.md`                        | The prompt. Interview → brief → playbook → base → mail plumbing → routines → go-live ladder.   |
| `SCHEMA.md`                         | The Airtable field contract: CRM tables plus the outreach control panel.                       |
| `PLAYBOOK_TEMPLATE.md`              | The Sales Playbook and Voice guide every writing routine reads on every run.                   |
| `workflows/mail-ingest.md`          | Mail and calendar capture into Activities (workflow engine).                                   |
| `workflows/mailer.md`               | The draft/send webhook, its contract, its sending switch, and the smoke test.                  |
| `routines/crm-reconciliation.md`    | The nightly CRM brain: classify, link, move status on evidence, digest.                        |
| `routines/reply-radar.md`           | Owed replies, replies to outreach, booked calls, weekly hooks. Draft-only.                     |
| `routines/outreach.md`              | One routine per sales motion. Pool, write, draft or send by Mode.                              |
| `routines/modules/prospect-pool.md` | Budget, exclusions, sourcing tiers, address ownership, scoring. Appended to outreach.          |
| `routines/modules/writing.md`       | Sources, format, never-say, self-check, the mailer call. Appended to outreach and Reply Radar. |
| `routines/modules/crm-writes.md`    | Who writes what, natural keys, the status ladder, audit trail. Appended to both.               |
| `MANUAL.md`                         | What a person must click, and the owner's switches.                                            |
| `OPERATORS.md`                      | The operator's manual template the agent fills in at the end.                                  |

Build order and value: CRM + ingest + reconciliation first, Reply Radar second, outreach last. Each stage is useful on its own.
