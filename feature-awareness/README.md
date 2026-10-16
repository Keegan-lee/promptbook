# Feature Awareness kit

An honest map of what your product actually does. Every feature, and every individual path through it (a page, an API route, a scheduled job, an email, a webhook, an agent tool), with evidence for each path in each environment. It answers five questions at a glance:

1. What does the product do? (the inventory)
2. What has been proven in production? (a real user, a real row, a real email)
3. What has only been walked in dev or staging?
4. What has never run anywhere?
5. What is high-risk and still at 0% production coverage?

An agent builds the inventory once from the codebase, pulls the first evidence from your databases and logs, and then refreshes the evidence every week. A person owns the sign-off: agents never write the four human fields. It is company-agnostic. The interview in `KICKOFF.md` makes it yours.

**To use it:** give your agent this folder plus `../_shared/`, and say "Run the Feature Awareness kickoff."

| File                           | What it is                                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `KICKOFF.md`                   | The prompt. Interview → brief → base → inventory from the codebase → first evidence pass → interface → routine → hand back.  |
| `SCHEMA.md`                    | The Airtable field contract: Features, Paths, Evidence Settings, the evidence-signal grammar, rollups, views, the interface. |
| `routines/evidence-refresh.md` | `FA · Evidence Refresh`: the weekly job that re-reads every reachable source and moves statuses on evidence only.            |
| `MANUAL.md`                    | What a person must click, the read-only access to grant, and the owner's switches.                                           |
| `OPERATORS.md`                 | The operator's manual template the agent fills in at the end: reading coverage, signing off, working the risk register.      |

Build order and value: the inventory alone is useful (most teams have never seen their product listed path by path). The first evidence pass makes it honest. The weekly routine keeps it honest. Sign-off makes it trusted.

## What the kit never does

- Writes to a product database. Every database is `SELECT` only, through a read-only role.
- Writes the four human sign-off fields: Tested by a human, Verified by, Date verified, Human notes.
- Guesses. A path no reachable source can see is `Unknown`, never `Never`. Absence of access is not absence of evidence.
- Moves a status backward. Evidence only ever adds proof; a person corrects a wrong status by hand.
- Copies personal data into Airtable. Evidence is counts, route names, table names, and test file paths.
