# Close checklist — template

> Build agent: tailor this with the person in KICKOFF Phase 3. Remove rows that do not apply, rename tasks into their words, set an owner and a reviewer for each, and move the due days to fit their calendar. Rows marked with a module disappear when that module is off. Write the tailored table into the brief (section "Close checklist", `Version 1.0`), then load one Close Task Templates row per line (`SCHEMA.md`). The brief is the source of truth: later changes go into the brief first, then into the table.

A close is a fixed list done in a fixed order. The executor prepares; a person records, approves, imports, and closes. A row whose **Prepared By** is `Agent` is work the routines do and a person reviews. A row whose **Prepared By** is `Person` is work only a person can do, usually inside the ledger or a bank.

**Due BD** counts business days after the period's last day, skipping the brief's holidays. Negative means before month end.

## Monthly

| Code | Task                                                               | Stage     | Module    | Prepared By | Owner (role) | Due BD | Evidence                                                                   | Auto check       | Turn off when                        |
| ---- | ------------------------------------------------------------------ | --------- | --------- | ----------- | ------------ | ------ | -------------------------------------------------------------------------- | ---------------- | ------------------------------------ |
| C01  | Remind submitters to send receipts and expense claims              | Pre-close | Close     | Person      | Close Owner  | -2     | Reminder sent                                                              | Manual           | nobody else spends company money     |
| C02  | Enter or approve every customer invoice for the month              | Pre-close | Close     | Person      | Close Owner  | 0      | Invoices dated in the period                                               | Manual           | no invoicing                         |
| C10  | Bank statements, every account                                     | Collect   | Close     | Person      | Close Owner  | 2      | One Documents row per account, Received                                    | File received    |                                      |
| C11  | Card statements, every card                                        | Collect   | Close     | Person      | Close Owner  | 2      | As above                                                                   | File received    | no company cards                     |
| C12  | Loan and credit-line statements                                    | Collect   | Close     | Person      | Close Owner  | 3      | As above                                                                   | File received    | no borrowing                         |
| C13  | Processor reports: balance, payouts, fees                          | Collect   | Revenue   | Person      | Close Owner  | 2      | One Documents row per Revenue Source, Received                             | File received    | Revenue module off and no processor  |
| C14  | Payroll reports and remittance statements                          | Collect   | Close     | Person      | Close Owner  | 3      | Payroll register, remittance confirmation                                  | File received    | no payroll                           |
| C15  | Ledger exports (see "Exports" below)                               | Collect   | Close     | Person      | Close Owner  | 3      | Every export in the list, dated after the last change                      | File received    | ledger connector covers every export |
| C20  | Bank and card feeds categorised in the ledger                      | Record    | Close     | Person      | Close Owner  | 4      | No uncategorised transactions in the GL export                             | Manual           |                                      |
| C21  | Missing receipts requested                                         | Record    | Close     | Agent       | Close Owner  | 4      | Documents rows for every expense at or above Receipt Threshold without one | File received    |                                      |
| C22  | Receivables aging reviewed                                         | Record    | Close     | Agent       | Close Owner  | 5      | Aging workpaper; questions for items past the brief's limit                | Manual           | no receivables                       |
| C23  | Vendor bills entered; payables aging reviewed                      | Record    | Close     | Agent       | Close Owner  | 5      | Aging workpaper                                                            | Manual           | no payables                          |
| C24  | Owner and shareholder expense claims recorded                      | Record    | Close     | Person      | Close Owner  | 5      | Claims in the ledger against the owner-balance account                     | Manual           | owners never pay company costs       |
| C30  | Bank reconciliations                                               | Reconcile | Close     | Agent       | Reviewer     | 6      | Reconciliations rows tie                                                   | Recon ties       |                                      |
| C31  | Card reconciliations                                               | Reconcile | Close     | Agent       | Reviewer     | 6      | As above                                                                   | Recon ties       | no company cards                     |
| C32  | Loan reconciliations and interest accrual                          | Reconcile | Close     | Agent       | Reviewer     | 6      | As above, plus the accrual entry proposed                                  | Recon ties       | no borrowing                         |
| C33  | Revenue to books: processor and product figures against the ledger | Reconcile | Revenue   | Agent       | Reviewer     | 5      | Revenue Recons rows Ties or explained                                      | Recon ties       | Revenue module off                   |
| C34  | Sales tax accounts reconciled                                      | Reconcile | Sales Tax | Agent       | Reviewer     | 7      | Tax account balances explained by returns filed, payments, current period  | Recon ties       | not registered                       |
| C35  | Payroll liabilities reconciled                                     | Reconcile | Close     | Agent       | Reviewer     | 7      | Liability balances equal amounts due next remittance                       | Recon ties       | no payroll                           |
| C36  | Owner and shareholder balances rolled forward                      | Reconcile | Close     | Agent       | Reviewer     | 7      | Opening + claims − repayments = closing, per person, in the workpaper      | Recon ties       | no owner balances                    |
| C37  | Suspense and uncategorised accounts at zero                        | Reconcile | Close     | Agent       | Reviewer     | 7      | Balance zero, or every item has a question                                 | Recon ties       |                                      |
| C38  | Prepaids and accruals match their schedules                        | Reconcile | Close     | Agent       | Reviewer     | 7      | Balances equal the Entry Schedules' remaining amounts                      | Recon ties       | no schedules                         |
| C39  | Amounts owed to third parties reconciled                           | Reconcile | Revenue   | Agent       | Reviewer     | 7      | Payable balance equals what the source says is owed                        | Recon ties       | company holds no one else's money    |
| C40  | Standing entries from Entry Schedules proposed                     | Adjust    | Close     | Agent       | Close Owner  | 6      | One Journal Entries row per active schedule                                | Manual           | no schedules                         |
| C41  | Accruals for costs incurred but not billed                         | Adjust    | Close     | Agent       | Close Owner  | 7      | Proposed entries or a note that none met Accrual Minimum                   | Manual           | cash-basis books                     |
| C42  | Deferred revenue and revenue timing                                | Adjust    | Revenue   | Agent       | Close Owner  | 7      | Proposed entries per the brief's revenue policy                            | Manual           | revenue earned when charged          |
| C43  | Depreciation and amortisation                                      | Adjust    | Close     | Agent       | Close Owner  | 7      | Proposed entries from schedules                                            | Manual           | nothing capitalised                  |
| C44  | Reclasses found in review                                          | Adjust    | Close     | Agent       | Close Owner  | 8      | Proposed entries, each with its evidence                                   | Manual           |                                      |
| C45  | Approve entries for import                                         | Adjust    | Close     | Person      | Owner        | 8      | No Proposed entries left; each approved or rejected by button              | Entries imported |                                      |
| C46  | Import the batch into the ledger and save the import result        | Adjust    | Close     | Person      | Close Owner  | 9      | Import Batches row Imported; result file in the batch folder               | Entries imported |                                      |
| C50  | Trial balance tie-out after import                                 | Review    | Close     | Agent       | Reviewer     | 9      | Fresh TB export equals expected balances, every account                    | Manual           |                                      |
| C51  | Movement review (flux) answered                                    | Review    | Close     | Agent       | Owner        | 9      | Every flux question answered                                               | Questions closed |                                      |
| C52  | Open questions answered or carried with a default                  | Review    | Close     | Person      | Owner        | 10     | No blocking questions open                                                 | Questions closed |                                      |
| C53  | Profit and loss and balance sheet read by the owner                | Review    | Close     | Person      | Owner        | 10     | Reports exported into the period folder                                    | File received    |                                      |
| C54  | Accountant review                                                  | Review    | Close     | Person      | Accountant   | 10     | The accountant's sign-off, saved in the period folder                      | File received    | accountant reviews only at year end  |
| C60  | Lock the period in the ledger                                      | Close     | Close     | Person      | Owner        | 10     | Ledger lock date on or after period end; Ledger Locked ticked              | Manual           | ledger has no lock (record that)     |
| C61  | Close the period                                                   | Close     | Close     | Person      | Owner        | 10     | "Close period" button                                                      | Manual           |                                      |

## Quarterly

Copied only into a month that ends a quarter (fiscal or calendar, as the brief says).

| Code | Task                                                             | Stage     | Module    | Prepared By | Owner (role) | Due BD | Evidence                                          | Auto check    | Turn off when               |
| ---- | ---------------------------------------------------------------- | --------- | --------- | ----------- | ------------ | ------ | ------------------------------------------------- | ------------- | --------------------------- |
| Q01  | Sales tax worksheets for returns due this quarter                | Quarterly | Sales Tax | Agent       | Reviewer     | 10     | Returns rows Worksheet Prepared                   | Manual        | not registered              |
| Q02  | Corporate tax instalment due, if any                             | Quarterly | Close     | Person      | Owner        | 10     | Returns row for the instalment Paid               | Manual        | no instalments              |
| Q03  | Fixed-asset and schedule review                                  | Quarterly | Close     | Agent       | Close Owner  | 10     | New capital purchases proposed as schedules       | Manual        | nothing capitalised         |
| Q04  | Quarterly report to the accountant (questions file, TB, entries) | Quarterly | Close     | Agent       | Close Owner  | 10     | Package in the period folder; the person sends it | File received | accountant only at year end |

## Annual

Copied into the fiscal year's last month. The Year-End Package routine does the work.

| Code | Task                                                        | Stage  | Module   | Prepared By | Owner (role) | Due BD       | Evidence                                          | Auto check    | Turn off when |
| ---- | ----------------------------------------------------------- | ------ | -------- | ----------- | ------------ | ------------ | ------------------------------------------------- | ------------- | ------------- |
| Y01  | Year-end package prepared                                   | Annual | Year-End | Agent       | Owner        | 30           | Package folder with its README and validation     | File received |               |
| Y02  | Year-end confirmations (loan balances, owner balances)      | Annual | Year-End | Person      | Owner        | 20           | Statements and signed confirmations in the folder | File received |               |
| Y03  | Inventory or work-in-progress count                         | Annual | Year-End | Person      | Owner        | 5            | Count sheet in the folder                         | File received | no inventory  |
| Y04  | Information slips and annual filings listed in the calendar | Annual | Year-End | Agent       | Owner        | 20           | Returns rows Worksheet Prepared                   | Manual        |               |
| Y05  | Corporate return worksheet approved and filed               | Annual | Year-End | Person      | Owner        | per calendar | Returns row Filed                                 | Manual        |               |

## Exports

The ledger exports a close needs, with the standard file name the routines give each one when they file it. Ask the person to export each one **after** the last change to the period, and again after every import.

| Export                                                           | Standard name                        | Used for                                        |
| ---------------------------------------------------------------- | ------------------------------------ | ----------------------------------------------- |
| Chart of accounts                                                | `<Period>_coa.csv`                   | Accounts mirror; import validation              |
| Trial balance at period end                                      | `<Period>_tb.csv`                    | Every reconciliation; tie-outs                  |
| General ledger detail (account transactions) for the period      | `<Period>_gl.csv`                    | Recon items, flux, missing receipts, duplicates |
| Journal list (manual journals) for the period and the one before | `<Period>_journals.csv`              | Duplicate check; import verification            |
| Receivables aging at period end                                  | `<Period>_ar_aging.csv`              | C22                                             |
| Payables aging at period end                                     | `<Period>_ap_aging.csv`              | C23                                             |
| Expenses or bills list with attachment flag                      | `<Period>_expenses.csv`              | Missing receipts                                |
| Payments received                                                | `<Period>_payments_received.csv`     | Deposits, revenue tie                           |
| Tax summary for the period                                       | `<Period>_tax_summary.csv`           | Sales tax accounts; worksheets                  |
| Profit and loss, balance sheet                                   | `<Period>_pl.pdf`, `<Period>_bs.pdf` | C53; the period's record                        |

When a connector can read a report, the routine reads it instead and saves a copy under the same name, so every period folder looks the same.

## Rules that do not change when you tailor

- A task is **Done** only by a person's button. The routines move tasks to Agent Prepared, Needs Review, Waiting on Docs, or Blocked.
- A reconciliation that does not tie is not hidden by a plug entry. Its difference stays visible until a reconciling item or a question explains it.
- Suspense and uncategorised accounts are zero at close, or every item in them has an open question.
- C45, C46, C60, and C61 are always a person. No setting moves them to the agent.
