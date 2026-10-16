# Accounting — schema

The field contract for the accounting shell. Build it with `_shared/AIRTABLE.md`. Substitute the brief's vocabulary wherever a select says **(brief)**. Leave out the tables of a module the brief turned off, and say so in the brief.

One base, `{{COMPANY}} Accounting`. It holds the work, not the books. The ledger is the books. The product database and the payment processor are the record of what customers were charged. This base tracks what has to be done, what was prepared, what a person approved, and what is still open.

| Group            | Tables                                                            | Module             |
| ---------------- | ----------------------------------------------------------------- | ------------------ |
| Control          | Settings, People, Run Log, Questions                              | always             |
| Close            | Periods, Close Task Templates, Close Tasks, Documents             | Monthly Close      |
| Ledger mirror    | Accounts                                                          | always             |
| Work             | Reconciliations, Journal Entries, Entry Schedules, Import Batches | Monthly Close      |
| Tax              | Tax Obligations, Returns                                          | always (see below) |
| Revenue to books | Revenue Sources, Revenue Recons                                   | Revenue to Books   |

Tax Obligations and Returns are built even when the Sales Tax module is off: the corporate return, instalments, and annual company filings live there too.

## What this base may and may not hold

- **May hold:** account-level figures (balances, totals, differences), file paths to workpapers, record ids from the ledger and the source systems, account nicknames, and the names of people who approve.
- **Never holds:** bank, card, loan, or processor account numbers; tax registration numbers; government ids of people or the company; passwords, API keys, or tokens; a person's pay.
- **Amounts live in figure fields and workpapers only.** Notes, summaries, the Run Log, and every chat message carry counts, statuses, and record ids.
- Share the base only with the people the brief names as finance users. The accountant gets an interface, not the base, unless the owner decides otherwise.

## Build order

1. Settings (scalar fields only) and People.
2. Accounts (from the ledger's chart-of-accounts export).
3. Periods, Close Task Templates (loaded from the tailored checklist), Close Tasks.
4. Documents.
5. Entry Schedules, Journal Entries, Import Batches, Reconciliations.
6. Tax Obligations (loaded from the confirmed obligations calendar), Returns.
7. Revenue Sources, Revenue Recons (module on only).
8. Questions, Run Log.
9. Links, created from one side only. Then lookups, counts, rollups. Formulas last.
10. Seed records: the one Settings row, People, Accounts, templates, obligations, revenue sources. Write the ID map into the brief, including the Settings record id.

Dates are ISO `YYYY-MM-DD`. Date-times use 24-hour time in `{{TIME_ZONE}}`; set the zone when you create each field, because the API cannot change it later. Currency fields use `{{BASE_CURRENCY}}` with the currency's minor units. A company that books in more than one currency adds a `Currency` single select next to every figure field and never sums across currencies.

## Vocabularies

Create every option before any routine or automation writes to the table. Routines never add options.

| Select               | Options                                                                                                                                                                                                     |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mode (each routine)  | Paused, Dry Run, Live                                                                                                                                                                                       |
| Ledger Access        | Exports only, Connector read-only, Connector with draft writes                                                                                                                                              |
| Routine              | AC · Monthly Close Prep, AC · Sales Tax Prep, AC · Revenue Reconciliation, AC · Year-End Package, Build, Person                                                                                             |
| Module               | Close, Sales Tax, Revenue, Year-End, Rebuild                                                                                                                                                                |
| Period Type          | Month, Quarter, Year                                                                                                                                                                                        |
| Period Status        | Not Started, Collecting, In Progress, In Review, Ready to Close, Closed, Reopened                                                                                                                           |
| Task Stage           | Pre-close, Collect, Record, Reconcile, Adjust, Review, Close, Quarterly, Annual                                                                                                                             |
| Frequency            | Monthly, Quarterly, Annual                                                                                                                                                                                  |
| Prepared By          | Agent, Person                                                                                                                                                                                               |
| Auto Check           | File received, Recon ties, Entries imported, Questions closed, Manual                                                                                                                                       |
| Task Status          | To Do, Waiting on Docs, Agent Prepared, Needs Review, Done, Not Applicable, Blocked                                                                                                                         |
| Account Type         | (brief) the ledger's own account types, verbatim from its chart-of-accounts export                                                                                                                          |
| Account Status       | Active, Inactive, Not in Ledger, Proposed                                                                                                                                                                   |
| Account Role         | Bank, Card, Loan, Clearing, Sales Tax Collected, Sales Tax Recoverable, Third-Party Payable, Owner Balance, Suspense, Deferred Revenue, Prepaid, Accrued Liability, Revenue, Contra Revenue, Processor Fees |
| Recon Type           | Bank, Card, Loan, Clearing, Sales Tax, Payroll Liabilities, Owner Balance, Prepaid, Accrual, Suspense, Third-Party Payable, Other                                                                           |
| Recon Status         | Not Started, Waiting on Statement, Prepared, Differences, Ties, Reviewed                                                                                                                                    |
| Doc Type             | Bank Statement, Card Statement, Loan Statement, Processor Report, Ledger Export, Receipt, Vendor Bill, Customer Invoice, Payroll Report, Tax Notice, Contract, Other                                        |
| Doc Status           | Expected, Requested, Received, Filed, Not Needed, Overdue                                                                                                                                                   |
| Entry Type           | Accrual, Reversal, Reclass, Correction, Prepaid Release, Depreciation, Deferred Revenue, Revenue Summary, Sales Tax Adjustment, Suspense Clearing, Opening Balance, Historical Rebuild                      |
| Entry Status         | Proposed, Needs Info, Approved for Import, In Import File, Imported, Import Failed, Rejected, Superseded                                                                                                    |
| Entry Source         | Close Prep, Revenue Recon, Sales Tax Prep, Year-End, Rebuild, Person                                                                                                                                        |
| Schedule Status      | Proposed, Active, Ended                                                                                                                                                                                     |
| Batch Kind           | Journals, Chart of Accounts, Opening Balances, Failed-only Re-run, Contacts                                                                                                                                 |
| Batch Status         | Prepared, Handed Over, Imported, Partly Failed, Failed, Verified in Ledger, Superseded                                                                                                                      |
| Validation           | Passed, Failed                                                                                                                                                                                              |
| Tax Type             | Sales Tax, Corporate Return, Tax Instalment, Payroll Remittance, Information Return, Annual Company Filing, Other (brief may add)                                                                           |
| Filing Method        | Authority portal, Paper, Accountant files, Through ledger, Other                                                                                                                                            |
| Weekend Rule         | Next business day, None, Confirm each date                                                                                                                                                                  |
| Return Status        | Upcoming, Data Pending, Worksheet Prepared, In Review, Approved to File, Filed, Paid, Not Required, Overdue                                                                                                 |
| Due Date Source      | Computed, Confirmed                                                                                                                                                                                         |
| Tie Out              | Ties, Does not tie, Not checked                                                                                                                                                                             |
| Source Kind          | Payment processor, Product database, Marketplace platform, App store, Invoicing system, Other                                                                                                               |
| Source Access        | Connector read-only, Read-only API key, Read-only database role, Exports                                                                                                                                    |
| Date Basis           | Created date, Settled date, Available date                                                                                                                                                                  |
| Revenue Model        | Gross (principal), Net (agent), Mixed: per product                                                                                                                                                          |
| Revenue Recon Status | Not Started, Data Pulled, Differences, Ties, Entries Proposed, Reviewed                                                                                                                                     |
| Asked Of             | Owner, Close Owner, Accountant, Other                                                                                                                                                                       |
| Question Status      | Open, Answered, Applied, Withdrawn                                                                                                                                                                          |
| Outcome              | OK, Quiet, Partial, Stopped, Error                                                                                                                                                                          |
| Error Status         | New, Investigating, Resolved, Ignored                                                                                                                                                                       |
| Person Role          | Owner, Close Owner, Bookkeeper, Accountant, Reviewer, Submitter                                                                                                                                             |

## Settings

Exactly one row, named `Accounting Settings`. Every routine reads it first and obeys it. The brief is the source of truth; this row is its live copy (`_shared/INTERVIEW.md`, rule 10). Primary field **Name**.

| Field                      | Type           | Notes and default                                                                                                                                          |
| -------------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name                       | single line    | `Accounting Settings`                                                                                                                                      |
| Close Prep Mode            | single select  | Paused                                                                                                                                                     |
| Sales Tax Mode             | single select  | Paused. Leave Paused when the module is off.                                                                                                               |
| Revenue Recon Mode         | single select  | Paused. Leave Paused when the module is off.                                                                                                               |
| Year-End Mode              | single select  | Paused                                                                                                                                                     |
| Ledger                     | single line    | The ledger's product name.                                                                                                                                 |
| Ledger Access              | single select  | From the brief.                                                                                                                                            |
| Ledger Draft Writes        | checkbox       | Off. Only `{{OWNER_NAME}}` turns it on. A guard may turn it off and write the Kill Reason.                                                                 |
| Target Period              | single line    | Empty = automatic. A period label (`YYYY-MM`) makes Close Prep and Revenue Recon work that period only. Used for the dry-run test and catch-up months.     |
| Target Return              | link → Returns | Empty = automatic. Makes Sales Tax Prep work that return only.                                                                                             |
| Open Period On BD          | number         | `{{OPEN_ON_BD \| 1}}`. Business day after month end on which Close Prep opens the period.                                                                  |
| Close Target BD            | number         | `{{CLOSE_TARGET_BD \| 10}}`                                                                                                                                |
| Close Window BD            | number         | `{{CLOSE_WINDOW_BD \| 15}}`. Close Prep does close work inside this window; outside it, only documents, due dates, and import results.                     |
| Recon Tolerance            | currency       | `{{RECON_TOLERANCE \| 0.00}}`. Bank, card, and loan reconciliations tie to the cent.                                                                       |
| Revenue Tolerance          | currency       | `{{REVENUE_TOLERANCE \| 1.00}}` per bucket, for rounding across many records.                                                                              |
| Flux Threshold %           | number         | `{{FLUX_PCT \| 25}}`                                                                                                                                       |
| Flux Minimum               | currency       | `{{FLUX_MIN}}` from the interview. A change must pass both to raise a question.                                                                            |
| Receipt Threshold          | currency       | `{{RECEIPT_THRESHOLD \| 0.00}}`. Expenses at or above it need a document. 0 means every expense.                                                           |
| Accrual Minimum            | currency       | `{{ACCRUAL_MIN}}` from the interview.                                                                                                                      |
| Tax Prep Lead BD           | number         | `{{TAX_PREP_LEAD_BD \| 15}}`. Sales Tax Prep starts a return this many business days before it is due.                                                     |
| Due Soon BD                | number         | `{{DUE_SOON_BD \| 10}}`                                                                                                                                    |
| Due Urgent BD              | number         | `{{DUE_URGENT_BD \| 3}}`                                                                                                                                   |
| Calendar Horizon Months    | number         | `{{CALENDAR_HORIZON \| 12}}`                                                                                                                               |
| Payout Match Days          | number         | `{{PAYOUT_MATCH_DAYS \| 5}}`                                                                                                                               |
| Sign-off Tolerance         | currency       | `{{SIGN_OFF_TOLERANCE \| 1.00}}`. Year-end cross-checks off by more than this stop the package.                                                            |
| Accounting Root            | single line    | The folder path every routine works inside. Nothing outside it is read or written.                                                                         |
| Inbox Path                 | single line    | `{{ACCOUNTING_ROOT}}/_inbox`                                                                                                                               |
| Import Columns             | long text      | The ledger's journal import header row, exact, from its own sample file. One line.                                                                         |
| Import Format              | long text      | Date format, decimal mark, sign convention (separate Debit/Credit or one signed Amount), account identifier (code, name, or both), status value for draft. |
| Max Lines Per File         | number         | `{{MAX_IMPORT_LINES \| 500}}`, or the ledger's own limit if lower.                                                                                         |
| Brief URL                  | url            |                                                                                                                                                            |
| Brief Version              | single line    |                                                                                                                                                            |
| Kill Reason                | long text      | Written by a guard that lowers a mode or a switch, or by the owner. Newest line on top.                                                                    |
| Last Close Prep At         | date-time      |                                                                                                                                                            |
| Last Close Prep Summary    | long text      | Counts and record ids only.                                                                                                                                |
| Last Sales Tax At          | date-time      |                                                                                                                                                            |
| Last Sales Tax Summary     | long text      |                                                                                                                                                            |
| Last Revenue Recon At      | date-time      |                                                                                                                                                            |
| Last Revenue Recon Summary | long text      |                                                                                                                                                            |
| Last Year-End At           | date-time      |                                                                                                                                                            |
| Last Year-End Summary      | long text      |                                                                                                                                                            |
| Last Changed By            | single line    | A person's name or a routine name.                                                                                                                         |

**Mode has three rungs.** Nothing in this kit sends, files, or publishes, so the short ladder applies (`_shared/ROUTINES.md`). **Dry Run:** the routine reads everything, computes, writes its workpapers and a `would-create` plan file under `{{ACCOUNTING_ROOT}}/_dry-run/`, and writes only its Run Log row and its own last-run fields. **Live:** it also creates and updates rows in this base and writes workpapers into the period folders. Neither rung touches the ledger, the processor, or a tax authority. Ledger draft writes are a separate owner switch, honoured only in Live.

## People

Everyone who appears in the work. Primary field **Name**.

| Field             | Type               | Notes                                              |
| ----------------- | ------------------ | -------------------------------------------------- |
| Name              | single line        |                                                    |
| Role              | multi select       | Person Role vocabulary.                            |
| Email             | email              |                                                    |
| Chat ID           | single line        | For DMs.                                           |
| Firm              | single line        | The accountant's firm, if any.                     |
| Receives Requests | checkbox           | May be asked for documents (receipts, statements). |
| Active            | checkbox           |                                                    |
| Documents         | link → Documents   | Inverse of Requested From.                         |
| Tasks Owned       | link → Close Tasks | Inverse of Owner.                                  |

## Accounts

A mirror of the ledger's chart of accounts, refreshed from each new export. It is how import files are validated before a person sees them. Primary field **Account Name**.

| Field               | Type                   | Notes                                                                                                                                         |
| ------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Account Name        | single line            | Exactly as the ledger spells it.                                                                                                              |
| Account Code        | single line            | Empty when the ledger has no codes.                                                                                                           |
| Account Key         | formula                | `IF({Account Code}, UPPER(TRIM({Account Code})), LOWER(TRIM({Account Name})))`. The natural key.                                              |
| Account Type        | single select          | The ledger's own type.                                                                                                                        |
| Parent Account      | single line            |                                                                                                                                               |
| Status              | single select          | Not in Ledger when a fresh export no longer lists it. Proposed for an account the kit wants created; it is created by a person in the ledger. |
| Role                | multi select           | What the account does in this kit.                                                                                                            |
| Nickname            | single line            | The name people use ("main operating account"). **Never a number.**                                                                           |
| Reconcile Monthly   | checkbox               |                                                                                                                                               |
| Recon Type          | single select          |                                                                                                                                               |
| Statement Source    | single line            | What proves the balance: "bank PDF statement", "processor balance report", "loan statement", "schedule".                                      |
| Journal Allowed     | checkbox               | The ledger accepts manual journals to it. Some ledgers block journals to bank, receivable, payable, or tax accounts.                          |
| Needs Contact       | checkbox               | The ledger requires a contact on journal lines to this account.                                                                               |
| Return Line         | single line            | The corporate-return line or classification code this account maps to.                                                                        |
| Return Line Status  | single select          | Proposed, Confirmed. Only the accountant or the owner confirms.                                                                               |
| Ledger Account Ref  | single line            | The ledger's internal id, when an export or connector exposes it.                                                                             |
| Last Seen In Export | date                   | Date of the newest chart-of-accounts export that listed it.                                                                                   |
| Reconciliations     | link → Reconciliations |                                                                                                                                               |
| Notes               | long text              |                                                                                                                                               |

## Periods

One row per accounting period. Primary field **Period**.

| Field              | Type                   | Notes                                                                                                                 |
| ------------------ | ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Period             | single line            | `YYYY-MM` for months, `YYYY-Qn` for quarters, `FYYYYY` for years (the year the fiscal year ends in). The natural key. |
| Period Type        | single select          |                                                                                                                       |
| Start Date         | date                   |                                                                                                                       |
| End Date           | date                   |                                                                                                                       |
| Fiscal Year        | single line            | `FYYYYY`                                                                                                              |
| Status             | single select          | Allowed moves below.                                                                                                  |
| Close Owner        | link → People          |                                                                                                                       |
| Target Close Date  | date                   | Written when the period opens: End Date plus Close Target BD, skipping the brief's holidays.                          |
| Folder             | single line            | `{{ACCOUNTING_ROOT}}/<FY>/<Period>`                                                                                   |
| Close Tasks        | link → Close Tasks     |                                                                                                                       |
| Tasks Open         | rollup                 | SUM of Close Tasks → Is Open. A 1/0 formula plus a plain SUM beats a conditional rollup (`_shared/AIRTABLE.md`).      |
| Reconciliations    | link → Reconciliations |                                                                                                                       |
| Recons Not Tied    | rollup                 | SUM of Reconciliations → Is Not Tied.                                                                                 |
| Journal Entries    | link → Journal Entries |                                                                                                                       |
| Entries Pending    | rollup                 | SUM of Journal Entries → Is Pending.                                                                                  |
| Documents          | link → Documents       |                                                                                                                       |
| Documents Missing  | rollup                 | SUM of Documents → Is Missing.                                                                                        |
| Questions          | link → Questions       |                                                                                                                       |
| Blocking Questions | rollup                 | SUM of Questions → Is Blocking Open.                                                                                  |
| Ledger Locked      | checkbox               | Ticked by a person after locking the period in the ledger.                                                            |
| Closed By          | collaborator           | Stamped by the "Close period" button automation. Never by an API write.                                               |
| Closed At          | date-time              | Same automation.                                                                                                      |
| Agent Notes        | long text              | Append-only, newest line on top, `YYYY-MM-DD · <routine>: <what> (<evidence>)`.                                       |

**Status moves.** `Not Started → Collecting → In Progress → In Review → Ready to Close` are the routine's to make, forward only, on the evidence in the Close Prep routine. `Closed` is a person's button, allowed only when Ledger Locked is ticked. `Closed → Reopened` is a person's move and is reported in every digest until the period is closed again.

## Close Task Templates

The tailored close checklist (`CLOSE_CHECKLIST_TEMPLATE.md`), one row per standard task. Close Prep copies the active rows into Close Tasks when it opens a period. Primary field **Task**.

| Field       | Type               | Notes                                                                                                        |
| ----------- | ------------------ | ------------------------------------------------------------------------------------------------------------ |
| Task        | single line        |                                                                                                              |
| Code        | single line        | `C10`, `Q01`, `Y01`. Unique. Part of the task's natural key.                                                 |
| Stage       | single select      |                                                                                                              |
| Frequency   | single select      | Quarterly tasks are copied only into a period that ends a quarter; Annual into the fiscal year's last month. |
| Module      | single select      |                                                                                                              |
| Prepared By | single select      | Agent or Person.                                                                                             |
| Owner       | link → People      |                                                                                                              |
| Reviewer    | link → People      |                                                                                                              |
| Due BD      | number             | Business days after period end. Negative means before.                                                       |
| Evidence    | long text          | What proves it is done.                                                                                      |
| Auto Check  | single select      | How the routine can tell the work is prepared.                                                               |
| Order       | number             |                                                                                                              |
| Active      | checkbox           |                                                                                                              |
| Close Tasks | link → Close Tasks |                                                                                                              |

## Close Tasks

One row per template per period. Primary field **Task**.

| Field       | Type                        | Notes                                                                                                                  |
| ----------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Task        | single line                 | `<Period> · <Code> · <task name>`                                                                                      |
| Task Key    | single line                 | `<Period>\|<Code>`. The natural key. Upsert on it.                                                                     |
| Period      | link → Periods              |                                                                                                                        |
| Template    | link → Close Task Templates |                                                                                                                        |
| Stage       | lookup                      | Template → Stage.                                                                                                      |
| Prepared By | lookup                      | Template → Prepared By.                                                                                                |
| Owner       | link → People               |                                                                                                                        |
| Status      | single select               | The routine may set Waiting on Docs, Agent Prepared, Needs Review, Blocked. Only a person sets Done or Not Applicable. |
| Due Date    | date                        |                                                                                                                        |
| Evidence    | long text                   | Paths and record ids.                                                                                                  |
| Done By     | collaborator                | Stamped by the "Done" button automation.                                                                               |
| Done At     | date-time                   | Same automation.                                                                                                       |
| Is Open     | formula                     | `IF(OR({Status} = "Done", {Status} = "Not Applicable"), 0, 1)`                                                         |
| Is Overdue  | formula                     | `IF(AND({Is Open}, {Due Date}, IS_BEFORE({Due Date}, TODAY())), 1, 0)`                                                 |
| Agent Notes | long text                   | Append-only.                                                                                                           |

## Documents

Every document the close needs, and every receipt the books are missing. Primary field **Document**.

| Field          | Type            | Notes                                                                                                       |
| -------------- | --------------- | ----------------------------------------------------------------------------------------------------------- |
| Document       | single line     | `<Period> · <Doc Type> · <account nickname or payee>`                                                       |
| Doc Number     | autonumber      |                                                                                                             |
| Document ID    | formula         | `"{{ID_PREFIX}}-D" & RIGHT("00000" & {Doc Number}, 5)`                                                      |
| Doc Key        | single line     | `<Period>\|<Doc Type>\|<Account Key or ledger transaction ref>`. The natural key.                           |
| Period         | link → Periods  |                                                                                                             |
| Doc Type       | single select   |                                                                                                             |
| Account        | link → Accounts | Statements only.                                                                                            |
| Requested From | link → People   |                                                                                                             |
| Status         | single select   | The routine moves Expected → Requested → Received → Filed, and sets Overdue. Only a person sets Not Needed. |
| Due Date       | date            |                                                                                                             |
| File Path      | single line     | Where the filed copy lives. Never a share link.                                                             |
| Received At    | date-time       |                                                                                                             |
| Ledger Txn Ref | single line     | Receipts only: the ledger's transaction id.                                                                 |
| Txn Date       | date            | Receipts only.                                                                                              |
| Payee          | single line     | Receipts only.                                                                                              |
| Amount         | currency        | Receipts only, so a person can find the receipt. Never copied into notes or chat.                           |
| Is Missing     | formula         | `IF(OR({Status} = "Expected", {Status} = "Requested", {Status} = "Overdue"), 1, 0)`                         |
| Notes          | long text       |                                                                                                             |

## Reconciliations

One row per account per period. Primary field **Recon**.

| Field             | Type             | Notes                                                                                                                    |
| ----------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Recon             | single line      | `<Period> · <account nickname>`                                                                                          |
| Recon Key         | single line      | `<Period>\|<Account Key>`. The natural key.                                                                              |
| Period            | link → Periods   |                                                                                                                          |
| Account           | link → Accounts  |                                                                                                                          |
| Recon Type        | single select    |                                                                                                                          |
| Source Balance    | currency         | From the statement or schedule. Sign convention: as the statement shows it (money held positive; amounts owed positive). |
| Source Read Twice | checkbox         | The closing balance was read from two places in the statement (summary and last running balance) and they agreed.        |
| Ledger Balance    | currency         | From the trial balance at period end, in the same sign convention.                                                       |
| Difference        | formula          | `{Source Balance} - {Ledger Balance}`                                                                                    |
| Reconciling Items | currency         | Sum of explained items (deposits in transit, outstanding payments, timing), from the workpaper.                          |
| Unexplained       | formula          | `{Difference} - {Reconciling Items}`                                                                                     |
| Tolerance         | currency         | Copied from Settings when the row is written.                                                                            |
| Is Not Tied       | formula          | `IF(ABS({Unexplained}) <= {Tolerance}, 0, 1)`                                                                            |
| Status            | single select    | The routine sets everything except Reviewed.                                                                             |
| Source Document   | single line      | Path to the statement.                                                                                                   |
| Workpaper         | single line      | Path to the reconciliation workpaper.                                                                                    |
| Prepared At       | date-time        |                                                                                                                          |
| Reviewed By       | collaborator     | Stamped by the "Reviewed" button automation.                                                                             |
| Reviewed At       | date-time        | Same automation.                                                                                                         |
| Questions         | link → Questions |                                                                                                                          |
| Agent Notes       | long text        | Append-only.                                                                                                             |

## Entry Schedules

Standing entries that repeat each period: prepaid releases, depreciation, deferred revenue releases, loan interest accruals, recurring accruals. A schedule becomes Active only when a person confirms it. Primary field **Schedule**.

| Field             | Type                   | Notes                                                                     |
| ----------------- | ---------------------- | ------------------------------------------------------------------------- |
| Schedule          | single line            |                                                                           |
| Entry Type        | single select          |                                                                           |
| Lines Template    | long text              | One line per journal line: `Dr\|Cr <account key> <amount> <description>`. |
| Amount Per Period | currency               |                                                                           |
| Start Period      | single line            | `YYYY-MM`                                                                 |
| End Period        | single line            | `YYYY-MM`                                                                 |
| Status            | single select          | Routines may create Proposed. Only a person sets Active or Ended.         |
| Confirmed By      | link → People          |                                                                           |
| Journal Entries   | link → Journal Entries |                                                                           |
| Notes             | long text              | The source: the invoice, the asset, the loan agreement (paths).           |

## Journal Entries

One row per prepared journal. The executor prepares; a person approves, imports, and posts. Primary field **Title**.

| Field            | Type                   | Notes                                                                                                                                                          |
| ---------------- | ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Title            | single line            | `<Period> · <Entry Type> · <short reason>`                                                                                                                     |
| Entry Number     | autonumber             |                                                                                                                                                                |
| JE Ref           | formula                | `"{{ID_PREFIX}}-J" & RIGHT("00000" & {Entry Number}, 5)`. Goes into the ledger's reference field on import, so a journal export proves what was imported.      |
| Entry Key        | single line            | `<Entry Source>\|<Period>\|<subject>` (for example `Close Prep\|2026-09\|schedule:<record id>`). The natural key. Upsert on it.                                |
| Period           | link → Periods         |                                                                                                                                                                |
| Journal Date     | date                   | Inside the period, and inside a period the ledger has not locked.                                                                                              |
| Entry Type       | single select          |                                                                                                                                                                |
| Entry Source     | single select          |                                                                                                                                                                |
| Reason           | long text              | Plain words a reviewer can check in a minute.                                                                                                                  |
| Evidence         | long text              | Paths and record ids. No amounts.                                                                                                                              |
| Lines            | long text              | One line per journal line: `Dr\|Cr <account key> <account name> <amount> <description>`.                                                                       |
| Line Count       | number                 |                                                                                                                                                                |
| Total Debits     | currency               | Computed in code, in minor units.                                                                                                                              |
| Total Credits    | currency               |                                                                                                                                                                |
| Balanced         | formula                | `IF(ROUND({Total Debits} - {Total Credits}, 2) = 0, "Balanced", "Out of balance")`                                                                             |
| Accounts Valid   | checkbox               | Every line's account exists in Accounts, is Active, and has Journal Allowed. Written by validation.                                                            |
| Validation Notes | long text              |                                                                                                                                                                |
| Status           | single select          | Routines set Proposed, Needs Info, In Import File, Imported, Import Failed, Superseded. **Approved for Import and Rejected come only from a person's button.** |
| Approved By      | collaborator           | Stamped by the "Approve for import" button automation.                                                                                                         |
| Approved At      | date-time              | Same automation.                                                                                                                                               |
| Import Batch     | link → Import Batches  |                                                                                                                                                                |
| Ledger Ref       | single line            | The ledger's journal number or id, once a journal export shows it.                                                                                             |
| Auto-Reverse On  | date                   | Accruals that reverse next period.                                                                                                                             |
| Reverses         | link → Journal Entries | The entry this one reverses.                                                                                                                                   |
| Schedule         | link → Entry Schedules |                                                                                                                                                                |
| Questions        | link → Questions       |                                                                                                                                                                |
| Is Pending       | formula                | `IF(OR({Status} = "Proposed", {Status} = "Needs Info", {Status} = "Approved for Import", {Status} = "In Import File", {Status} = "Import Failed"), 1, 0)`      |
| Agent Notes      | long text              | Append-only.                                                                                                                                                   |

## Import Batches

One row per file handed to a person to import. A file is never edited after hand-over; a correction is a new version. Primary field **Batch**.

| Field              | Type                   | Notes                                                                                  |
| ------------------ | ---------------------- | -------------------------------------------------------------------------------------- |
| Batch              | single line            | `<Batch ID> · <Period> · <Kind> · v<Version>`                                          |
| Batch Number       | autonumber             |                                                                                        |
| Batch ID           | formula                | `"{{ID_PREFIX}}-B" & RIGHT("0000" & {Batch Number}, 4)`                                |
| Period             | link → Periods         |                                                                                        |
| Kind               | single select          |                                                                                        |
| Version            | number                 |                                                                                        |
| File Path          | single line            |                                                                                        |
| Journal Entries    | link → Journal Entries |                                                                                        |
| Journal Count      | number                 |                                                                                        |
| Line Count         | number                 |                                                                                        |
| Total Debits       | currency               |                                                                                        |
| Total Credits      | currency               |                                                                                        |
| Validation         | single select          |                                                                                        |
| Validation Notes   | long text              | Which checks ran and passed.                                                           |
| Status             | single select          | The routine sets every status except Imported, which is the person's button.           |
| Parent Batch       | link → Import Batches  | A failed-only re-run points at the batch it repairs.                                   |
| Import Result Path | single line            | The ledger's import report or error file, saved by the person into the batch's folder. |
| Imported By        | collaborator           | Stamped by the "Imported" button automation.                                           |
| Imported At        | date-time              | Same automation.                                                                       |
| Verified At        | date-time              | When a fresh journal export showed every JE Ref in the batch.                          |
| Agent Notes        | long text              | Append-only.                                                                           |

## Tax Obligations

The obligations calendar: one row per recurring filing or payment. Built from the interview and **confirmed by the owner or the accountant** before any routine uses it. Primary field **Obligation**.

| Field              | Type            | Notes                                                                                                                                                        |
| ------------------ | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Obligation         | single line     | Plain name, for example "Sales tax return — home region".                                                                                                    |
| Obligation Code    | single line     | Short and unique (`ST-HOME-Q`, `CORP-RETURN`). Part of every return's key.                                                                                   |
| Tax Type           | single select   |                                                                                                                                                              |
| Authority          | single line     | The authority's name.                                                                                                                                        |
| Registration Label | single line     | A nickname for the registration. **Never the registration number.**                                                                                          |
| Frequency          | single select   |                                                                                                                                                              |
| Filing Due Months  | number          | Months after period end.                                                                                                                                     |
| Filing Due Day     | number          | Day of that month; 0 means the last day.                                                                                                                     |
| Payment Due Months | number          | Often differs from filing. Ask.                                                                                                                              |
| Payment Due Day    | number          |                                                                                                                                                              |
| Weekend Rule       | single select   | "Confirm each date" when nobody knows.                                                                                                                       |
| Filing Method      | single select   |                                                                                                                                                              |
| Files It           | link → People   | Who files. Never the executor.                                                                                                                               |
| Pays It            | link → People   | Who pays. Never the executor.                                                                                                                                |
| Form Lines         | long text       | The return's lines as the authority labels them, one per line, confirmed. Worksheets use exactly these labels.                                               |
| Rates              | long text       | `<rate> · <jurisdiction or category> · from <date> · to <date or open> · source <notice/accountant>`. Confirmed. Routines never look a rate up and apply it. |
| Accounts           | link → Accounts | The ledger's tax accounts for this obligation.                                                                                                               |
| Confirmed By       | link → People   |                                                                                                                                                              |
| Confirmed At       | date            |                                                                                                                                                              |
| Source             | long text       | Where the rule came from: the registration letter, the authority's notice, the accountant. Paths only.                                                       |
| Module             | single select   |                                                                                                                                                              |
| Active             | checkbox        |                                                                                                                                                              |
| Returns            | link → Returns  |                                                                                                                                                              |

## Returns

One row per obligation per period. Primary field **Return**.

| Field              | Type                   | Notes                                                                                                                                            |
| ------------------ | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Return             | single line            | `<Obligation Code> · <period label>`                                                                                                             |
| Return Key         | single line            | `<Obligation Code>\|<Period End>`. The natural key.                                                                                              |
| Obligation         | link → Tax Obligations |                                                                                                                                                  |
| Period Label       | single line            |                                                                                                                                                  |
| Period Start       | date                   |                                                                                                                                                  |
| Period End         | date                   |                                                                                                                                                  |
| Filing Due         | date                   |                                                                                                                                                  |
| Payment Due        | date                   |                                                                                                                                                  |
| Due Date Source    | single select          | Computed until a person confirms it against the authority's account or notice.                                                                   |
| Status             | single select          | Routines set Upcoming, Data Pending, Worksheet Prepared, In Review, Overdue. **Approved to File, Filed, Paid, Not Required come from a person.** |
| Preliminary        | checkbox               | The worksheet was prepared before every month in the return period was closed.                                                                   |
| Worksheet Path     | single line            |                                                                                                                                                  |
| Worksheet Version  | number                 |                                                                                                                                                  |
| Tie Out            | single select          | Worksheet totals against the ledger's tax accounts.                                                                                              |
| Approved By        | collaborator           | "Approve worksheet" button.                                                                                                                      |
| Approved At        | date-time              |                                                                                                                                                  |
| Filed By           | collaborator           | "Mark filed" button, clicked by the person who filed.                                                                                            |
| Filed At           | date-time              |                                                                                                                                                  |
| Confirmation File  | single line            | Path to the authority's confirmation, saved by the person. Not the confirmation number itself.                                                   |
| Paid               | checkbox               | Ticked by the person who paid.                                                                                                                   |
| Days To Filing Due | formula                | `IF({Filing Due}, DATETIME_DIFF({Filing Due}, TODAY(), 'days'), BLANK())`                                                                        |
| Questions          | link → Questions       |                                                                                                                                                  |
| Agent Notes        | long text              | Append-only.                                                                                                                                     |

## Revenue Sources

One row per system that says what the company earned: a payment processor, the product's own database, a marketplace platform, an app store, an invoicing tool. Primary field **Source**.

| Field                   | Type                  | Notes                                                                                                                        |
| ----------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Source                  | single line           | A nickname. Never an account number.                                                                                         |
| Kind                    | single select         |                                                                                                                              |
| Access                  | single select         | Read-only only. A key that can refund, pay out, or transfer is refused at build.                                             |
| Currency                | single line           | One row per currency when a source settles in several.                                                                       |
| Date Basis              | single select         | Which date puts a record in a period. Must match how the ledger books it.                                                    |
| Revenue Model           | single select         | Policy, confirmed by the accountant or owner. The routine never decides it.                                                  |
| Credits Funded By       | long text             | Per credit or discount programme: who pays for it (the company, a sponsor, a partner) and how the brief says to book it.     |
| Fee Waiver Rule         | long text             | Which records carry a waived fee, and the field that says so.                                                                |
| Clearing Account        | link → Accounts       | The ledger account that mirrors the processor balance.                                                                       |
| Revenue Accounts        | link → Accounts       |                                                                                                                              |
| Contra Revenue Accounts | link → Accounts       | Refunds, discounts, credits, as the brief books them.                                                                        |
| Fee Account             | link → Accounts       |                                                                                                                              |
| Third-Party Payable     | link → Accounts       | Amounts owed to sellers, hosts, or partners.                                                                                 |
| Tax Accounts            | link → Accounts       |                                                                                                                              |
| Bank Account            | link → Accounts       | Where payouts land.                                                                                                          |
| Bucket Map              | long text             | For each bucket (below), the exact report column or database field and filter that produces it. Written at build, versioned. |
| Bucket Map Version      | single line           |                                                                                                                              |
| Active                  | checkbox              |                                                                                                                              |
| Revenue Recons          | link → Revenue Recons |                                                                                                                              |

The buckets: Opening Balance, Gross Charges, Refunds, Disputes, Credits and Discounts, Fee Waivers, Processor Fees, Third-Party Share, Tax Collected, Payouts, Adjustments, Closing Balance.

## Revenue Recons

One row per source per period. The figures on the source side come from the source; the ledger side comes from the ledger; this row never makes them agree. Primary field **Run**.

| Field                       | Type                   | Notes                                                                                       |
| --------------------------- | ---------------------- | ------------------------------------------------------------------------------------------- |
| Run                         | single line            | `<Period> · <Source>`                                                                       |
| Run Key                     | single line            | `<Period>\|<Source record id>`. The natural key.                                            |
| Period                      | link → Periods         |                                                                                             |
| Source                      | link → Revenue Sources |                                                                                             |
| Bucket Map Version          | single line            | The map the run used.                                                                       |
| Gross Charges               | currency               | Source side.                                                                                |
| Refunds                     | currency               | Source side.                                                                                |
| Disputes                    | currency               | Source side.                                                                                |
| Credits and Discounts       | currency               | Source side.                                                                                |
| Fee Waivers                 | currency               | Source side. The fee the company did not charge.                                            |
| Processor Fees              | currency               | Source side.                                                                                |
| Third-Party Share           | currency               | Source side. Owed to others.                                                                |
| Tax Collected               | currency               | Source side.                                                                                |
| Payouts                     | currency               | Source side.                                                                                |
| Source Closing Balance      | currency               | Source side.                                                                                |
| Company Revenue (Source)    | currency               | Computed in code from per-record fields under the Revenue Model. Never from a default rate. |
| Ledger Revenue              | currency               |                                                                                             |
| Ledger Fees                 | currency               |                                                                                             |
| Ledger Third-Party Movement | currency               |                                                                                             |
| Ledger Tax                  | currency               |                                                                                             |
| Ledger Clearing Balance     | currency               |                                                                                             |
| Payouts Unmatched           | number                 | Payouts with no bank deposit inside Payout Match Days.                                      |
| Differences                 | number                 | Count of buckets outside Revenue Tolerance.                                                 |
| Unexplained Buckets         | number                 | Count left after timing and known items.                                                    |
| Source Rolls Forward        | checkbox               | Opening + activity − payouts = closing, on the source's own figures.                        |
| Status                      | single select          | The routine sets everything except Reviewed.                                                |
| Workpaper                   | single line            | Path to the bridge.                                                                         |
| Journal Entries             | link → Journal Entries | Proposed entries this run raised.                                                           |
| Questions                   | link → Questions       |                                                                                             |
| Reviewed By                 | collaborator           | "Reviewed" button.                                                                          |
| Reviewed At                 | date-time              |                                                                                             |
| Agent Notes                 | long text              | Append-only.                                                                                |

## Questions

Open questions for the owner or the accountant. The executor asks; a person answers. Primary field **Question**.

| Field             | Type           | Notes                                                                                                                                                      |
| ----------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Question          | long text      | One question, answerable in a sentence.                                                                                                                    |
| Question Number   | autonumber     |                                                                                                                                                            |
| Question ID       | formula        | `"{{ID_PREFIX}}-Q" & RIGHT("0000" & {Question Number}, 4)`                                                                                                 |
| Question Key      | single line    | `<Module>\|<Period or year>\|<subject ref>`. The natural key: one question per subject.                                                                    |
| Period            | link → Periods |                                                                                                                                                            |
| Module            | single select  |                                                                                                                                                            |
| Asked Of          | single select  |                                                                                                                                                            |
| Why It Matters    | long text      | Which figure, return, or entry depends on the answer.                                                                                                      |
| Default Treatment | long text      | What the work assumes until answered, or `none: blocks`. Defaults are conservative: no credit claimed, no revenue recognised early, item held in suspense. |
| Blocking          | checkbox       |                                                                                                                                                            |
| Decision          | long text      | Typed by a person. Routines never write it.                                                                                                                |
| Status            | single select  | Routines set Open, and Applied after acting on a Decision. A person sets Answered or Withdrawn.                                                            |
| Answered By       | collaborator   | Stamped by the "Answered" button automation.                                                                                                               |
| Answered At       | date-time      |                                                                                                                                                            |
| Raised By         | single select  | Routine vocabulary.                                                                                                                                        |
| Raised At         | date           |                                                                                                                                                            |
| Is Blocking Open  | formula        | `IF(AND({Blocking}, OR({Status} = "Open", {Status} = "Answered")), 1, 0)`                                                                                  |
| Agent Notes       | long text      | Append-only.                                                                                                                                               |

Links from Reconciliations, Journal Entries, Returns, and Revenue Recons land here as their inverses.

## Run Log

One row per routine run. Primary field **Run**.

| Field            | Type          | Notes                                                                    |
| ---------------- | ------------- | ------------------------------------------------------------------------ |
| Run              | single line   | `<routine> · <YYYY-MM-DD HH:mm>` in the home time zone.                  |
| Routine          | single select |                                                                          |
| Started At       | date-time     |                                                                          |
| Mode             | single select | The mode the run used.                                                   |
| Period           | single line   | The period or return it worked.                                          |
| Outcome          | single select |                                                                          |
| Rows Created     | number        |                                                                          |
| Rows Updated     | number        |                                                                          |
| Files Written    | number        |                                                                          |
| Entries Proposed | number        |                                                                          |
| Questions Raised | number        |                                                                          |
| Summary          | long text     | Counts, statuses, record ids. **No amounts, names, or account numbers.** |
| Error Message    | long text     | One line. No tokens, headers, file contents, or figures.                 |
| Error Status     | single select | New on an Error row; people move it on.                                  |

## Views

The API cannot create filtered views. They are in `MANUAL.md` for a person.

| Table           | View              | Filter / sort                                               |
| --------------- | ----------------- | ----------------------------------------------------------- |
| Periods         | All               | none. **Do not filter or delete:** automations watch it.    |
| Periods         | Open              | Status not Closed, sorted by End Date                       |
| Close Tasks     | This Close        | Period = the open period, grouped by Stage, sorted by Order |
| Close Tasks     | Overdue           | Is Overdue = 1                                              |
| Documents       | Missing           | Is Missing = 1, grouped by Requested From                   |
| Reconciliations | Not Tied          | Is Not Tied = 1                                             |
| Reconciliations | To Review         | Status Ties or Differences, Reviewed By empty               |
| Journal Entries | To Approve        | Status Proposed, Balanced = Balanced, Accounts Valid ticked |
| Journal Entries | Needs Info        | Status Needs Info or Import Failed                          |
| Journal Entries | All               | none. **Do not filter or delete.**                          |
| Import Batches  | Waiting on Import | Status Prepared or Handed Over                              |
| Returns         | Calendar          | Status not Filed/Paid/Not Required, sorted by Filing Due    |
| Returns         | To Approve        | Status Worksheet Prepared or In Review                      |
| Revenue Recons  | Not Tied          | Unexplained Buckets > 0 or Payouts Unmatched > 0            |
| Questions       | Open — Owner      | Status Open, Asked Of Owner or Close Owner, Blocking first  |
| Questions       | Open — Accountant | Status Open, Asked Of Accountant                            |
| Run Log         | Errors            | Outcome Error, Error Status New or Investigating            |

## Interfaces

**Close Desk** (the owner's and close owner's workspace):

| Page            | Type      | Source and buttons                                                                                                                                  |
| --------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Control Panel   | dashboard | The Settings record (modes, Ledger Draft Writes, tolerances, Kill Reason, last-run summaries), Open periods, Returns calendar for the next 90 days. |
| This Close      | list      | Close Tasks, This Close view. Button **Done**.                                                                                                      |
| Documents       | list      | Documents, Missing view. Editable File Path and Status (Not Needed).                                                                                |
| Reconciliations | list      | Not Tied and To Review. Detail shows the Workpaper path. Button **Reviewed**.                                                                       |
| Entries         | list      | To Approve. Detail shows Reason, Evidence, Lines, Balanced, Accounts Valid. Buttons **Approve for import** and **Reject**.                          |
| Imports         | list      | Waiting on Import. Detail shows File Path and Validation Notes. Button **Imported**.                                                                |
| Returns         | list      | Calendar and To Approve. Buttons **Approve worksheet**, **Mark filed**. Editable Confirmation File and Paid.                                        |
| Revenue         | list      | Revenue Recons, newest first. Button **Reviewed**. (Module on only.)                                                                                |
| Questions       | list      | Open — Owner. Editable Decision. Button **Answered**.                                                                                               |
| Period          | detail    | One period with its tasks, recons, entries, documents, and questions. Button **Close period** (shown only when Ledger Locked is ticked).            |

**Accountant View** (optional, read-mostly): Questions Open — Accountant with editable Decision and the Answered button; Journal Entries for the period; Reconciliations; Returns; Year-End package paths. Shared by the owner only.

## Automations

Created as drafts. A person turns each on after a test record proves it (`_shared/AIRTABLE.md`, approval stamps).

| Name                | Trigger                                         | Action                                                                                 | Default                   |
| ------------------- | ----------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------- |
| Approve entry       | Button "Approve for import" on Journal Entries  | Status = Approved for Import, Approved By = the person who clicked, Approved At = now. | on                        |
| Reject entry        | Button "Reject"                                 | Status = Rejected, stamp as above.                                                     | on                        |
| Recon reviewed      | Button "Reviewed" on Reconciliations            | Status = Reviewed, Reviewed By, Reviewed At.                                           | on                        |
| Revenue reviewed    | Button "Reviewed" on Revenue Recons             | Status = Reviewed, Reviewed By, Reviewed At.                                           | on                        |
| Batch imported      | Button "Imported" on Import Batches             | Status = Imported, Imported By, Imported At.                                           | on                        |
| Task done           | Button "Done" on Close Tasks                    | Status = Done, Done By, Done At.                                                       | on                        |
| Question answered   | Button "Answered" on Questions                  | Status = Answered, Answered By, Answered At. Refuses when Decision is empty.           | on                        |
| Approve worksheet   | Button "Approve worksheet" on Returns           | Status = Approved to File, Approved By, Approved At.                                   | on                        |
| Mark filed          | Button "Mark filed" on Returns                  | Status = Filed, Filed By, Filed At.                                                    | on                        |
| Close period        | Button "Close period" on Periods                | Only when Ledger Locked is ticked: Status = Closed, Closed By, Closed At.              | on                        |
| Settings change log | Any Mode, or Ledger Draft Writes, changes       | Email `{{OWNER_EMAIL}}` the field, old and new value, and Last Changed By.             | on                        |
| Run error email     | Run Log row created with Outcome Error          | Email `{{OWNER_EMAIL}}` the Run, the Routine, and the one-line Error Message.          | on                        |
| Receipt reminder    | Documents row becomes Overdue, Doc Type Receipt | Email the Requested From person the Document ID and Txn Date. No amount.               | off unless the brief asks |

Every button automation is the only way its status and stamp get set. Routines treat a record as approved only when **both** the status and its stamp are present (`LEN({Approved By} & '') > 0`); a status without a stamp is reported as "unstamped approval" and ignored. Test each one: click it in the interface and see your own stamp; then have the agent set the same status through the API on a `[TEST]` record and confirm no stamp appears and the routines treat it as unapproved. Delete the test records afterwards.
