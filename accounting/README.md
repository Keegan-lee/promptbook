# Accounting kit

A portable accounting pipeline with Airtable as the shell: a monthly close that prepares itself, sales tax worksheets tied to the ledger, a bridge from what the product and payment processor say the company earned to what the books show, and a year-end package for the accountant. It is company-agnostic, ledger-agnostic, and jurisdiction-agnostic. The interview in `KICKOFF.md` makes it yours and decides which modules are on.

**It prepares. People file and pay.** The executor is not a licensed accountant. It prepares workpapers, reconciliations, journal-entry import files, and return worksheets for a person, and their accountant if they have one, to review, import, file, and pay. It never posts a journal, files a return, moves money, or types an account number, a tax id, or a password.

**To use it:** give your agent this folder plus `../_shared/`, and say "Run the Accounting kickoff."

| File                                 | What it is                                                                                                                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `KICKOFF.md`                         | The prompt. Interview → brief → calendar, checklist, and policies → base → folders and ledger plumbing → routines → proof on known-good books. Ends with jurisdiction prompts for the interview. |
| `SCHEMA.md`                          | The Airtable field contract: settings and modes, periods, close tasks, documents, accounts, reconciliations, entries, import batches, obligations, returns, revenue sources, questions, run log. |
| `CLOSE_CHECKLIST_TEMPLATE.md`        | The monthly, quarterly, and annual checklist the interview tailors, and the exports a close needs.                                                                                               |
| `routines/monthly-close-prep.md`     | `AC · Monthly Close Prep`: opens the period, files statements, reconciles, proposes entries, builds and verifies import files, watches due dates.                                                |
| `routines/revenue-reconciliation.md` | `AC · Revenue Reconciliation`: processor and product figures against the ledger through clearing accounts; every difference classified.                                                          |
| `routines/sales-tax-prep.md`         | `AC · Sales Tax Prep`: one worksheet per registration and period, every line VERIFIED, CHECK, or FILL BY HAND.                                                                                   |
| `routines/year-end-package.md`       | `AC · Year-End Package` (manual): year-end adjustments, statements, tax package, accountant package; or a phased catch-up rebuild.                                                               |
| `MANUAL.md`                          | What a person must click, the owner's switches, and what only a person ever does.                                                                                                                |
| `OPERATORS.md`                       | The operator's manual template the agent fills in at the end.                                                                                                                                    |

## Modules

| Module           | On when                                                    | Needs                                                      |
| ---------------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| Monthly Close    | always                                                     | the ledger (connector or exports), statements, a folder    |
| Sales Tax        | the company is registered to collect a sales tax           | a confirmed obligations calendar with form lines and rates |
| Revenue to Books | the company takes money through its product or a processor | read-only processor and product access, clearing accounts  |
| Year-End         | always (Rebuild path when the books are behind)            | the last filed return                                      |

## Two systems of record

The product database and the payment processor record what customers were charged, refunded, credited, waived, and paid out, and what is owed to others. The ledger records the company's accounts. The kit reconciles them and lists the differences. It never changes either side to make them agree, and reports about one never borrow the other's numbers.

## Order and value

Monthly Close first, proved on a month the person already closed. Revenue to Books next, if the company sells through its own product: it catches the errors that booking payouts as revenue hides. Sales Tax before the next return is due. Year-End once a year.
