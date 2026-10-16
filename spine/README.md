# SPINE kit (Company State)

The **SPINE** is one short, dated document that says what is true about the company right now: the current numbers, the live deadlines, the open items, who is who. Every agent and every person reads it first. This kit builds it, keeps it truthful every week, and gives it a memory of _why_.

It is company-agnostic. The interview in `KICKOFF.md` makes it yours.

**To use it:** give your agent this folder plus `../_shared/`, and say "Run the SPINE kickoff."

## How it works

```
company systems ──(read-only)──┐
  Data Room · task tracker ·   │
  CRM · product database …     ▼
                       CS · SPINE Sync (weekly)
                         gather → classify → compose → create → read back → archive prior → pointer last
                         │                   │                                         │
                         ▼                   ▼                                         ▼
     Observations (audit record,   SPINE_<date> in the SPINE folder     Articles: the pointer record
     one per open conflict)        prior versions in the archive        (every agent starts here)
                         │
                         └──▶ owner's DM: only what needs the owner

shared-drive inbox ──▶ CS · Drive Filing (weekly, optional) ──▶ department folders
                                                       └──▶ Data Room candidates + manual-task list
```

- **What moves stays in the SPINE.** Context that has solidified is extracted to its own document, registered in the Data Room as an Article, and cited from the SPINE in one line. The SPINE stays under its word ceiling by extraction, never by cutting.
- **The Data Room is the index and the memory of why.** Articles (every document), Decisions (why the company is shaped this way, including what was rejected), Observations (raw notes, cheap to log), and optionally Board Minutes.
- **The sync never settles a real conflict.** When two systems of record disagree on something that changes a number, a right, an obligation, or a commitment to a third party, it carries the old wording forward with an `[UNRECONCILED <date> — see Observation <id>]` marker and asks the owner.

## Files

| File                         | What it is                                                                                                            |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `KICKOFF.md`                 | The prompt. Interview → brief → Data Room → folders → protocol and first SPINE → routines → dry run → hand-back.      |
| `SCHEMA.md`                  | The Data Room base, the settings table, the pointer record, and where SPINE files live.                               |
| `SPINE_TEMPLATE.md`          | The SPINE document skeleton: header, §0 pointer to the protocol, standing facts, the sections that move, the log.     |
| `AGENT_PROTOCOL_TEMPLATE.md` | How every agent works here: read the SPINE first, the authority order, how to cite, what never goes in the Data Room. |
| `routines/spine-sync.md`     | `CS · SPINE Sync`, weekly. The only writer of the SPINE besides the owner.                                            |
| `routines/drive-filing.md`   | `CS · Drive Filing`, weekly, optional. Files the inbox folder, never deletes, flags Data Room candidates.             |
| `MANUAL.md`                  | What a person must click, and the owner's switches.                                                                   |
| `OPERATORS.md`               | The operator's manual template the agent fills in at the end.                                                         |

Build order and value: the Data Room and the agent protocol first, the first SPINE second (composed with the owner), the weekly sync third, Drive Filing last. Each stage is useful on its own.

## Using the SPINE from other kits

Every routine in every kit can read the SPINE. It needs read access to the Data Room base and to the SPINE folder, nothing else.

1. In that kit's brief, set `{{COMPANY_STATE_DOC}}` (the sales kit already has it in Block E) to this exact text, filled from this kit's ID map:

   > the SPINE: read Data Room base `{{DATA_ROOM_BASE_ID}}`, Articles record `{{POINTER_RECORD_ID}}`, open its Drive Link, and read to the line `END OF SPINE`. Never use a pinned file id. Never read the SPINE Archive folder.

2. The routine then follows the authority order the SPINE states at its top. The sales kit's reconciliation already places "company state document" between "signed agreement" and "current system state", with the customer system outranking it for transaction, revenue, and usage facts. That matches the default here.
3. A routine that finds the SPINE more than `{{STALE_AFTER_DAYS | 8}}` days past its Last verified date says "SPINE stale" in its run summary and trusts current system state for anything time-sensitive.
4. No routine outside this kit writes the SPINE, the pointer record, or the archive. If another routine finds the SPINE wrong, it logs one Observation (Author `Agent`) and moves on. The next sync picks it up.
5. For agents that work interactively (a coding agent, a research session), point them at the **Agent Operating Protocol** the kickoff creates from `AGENT_PROTOCOL_TEMPLATE.md`. It is registered in the Data Room and cited from SPINE §0.

## Start small

The smallest version that pays for itself: a Data Room with Articles, Decisions, and Observations, a first SPINE the owner approves, and the pointer record. Run the sync by hand for two weeks before you schedule it. Add Drive Filing only when the shared drive's inbox folder is actually used.
