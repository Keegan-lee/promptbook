# Accounting — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the owner, the close owner, and the reviewers. Replace every `{{…}}`, drop the sections and the table rows marked for a module the brief turned off (and the module labels on the rows you keep), and add the live links.

Live base: [{{COMPANY}} Accounting]({{URL_BASE}}). Day-to-day work happens in the **Close Desk** interface ({{URL_CLOSE_DESK}}). Files live in `{{ACCOUNTING_ROOT}}`.

**Everything this system prepares is a draft for review.** It is not a licensed accountant. It never posts, files, pays, or moves money. {{#if ACCOUNTANT}}When {{ACCOUNTANT_NAME}} disagrees with a worksheet, {{ACCOUNTANT_NAME}} is right.{{/if}}

## Your month

| When                         | What arrives                                        | What you do                                                                                                                                                                                                |
| ---------------------------- | --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Last two business days       | Close checklist reminder (C01)                      | Remind everyone to submit receipts and claims.                                                                                                                                                             |
| BD 1–3                       | Close digest: documents expected                    | Drop every statement and ledger export into `_inbox`. Any file name works.                                                                                                                                 |
| BD 2–5                       | Missing receipts, unrecognised files                | Attach receipts in {{LEDGER}}. Categorise the bank and card feeds.                                                                                                                                         |
| BD 2–6 (Revenue module)      | Revenue digest                                      | Read the differences; answer the questions.                                                                                                                                                                |
| BD 5–8                       | Entries to approve, reconciliations to review       | Open **Entries**. Read Reason, Evidence, Lines. Approve or reject each. Press **Reviewed** on each reconciliation you checked.                                                                             |
| The day after you approve    | An import file in `imports/`                        | Import it into {{LEDGER}} ({{LEDGER_IMPORT_STEPS}}). Save the ledger's import result next to the file. Press **Imported**. Export the trial balance, general ledger, and journal list again into `_inbox`. |
| BD 9–10                      | Tie-out result, questions                           | Answer the open questions. Read the profit and loss and balance sheet.                                                                                                                                     |
| BD {{CLOSE_TARGET_BD \| 10}} | "Ready to Close" in the digest                      | Lock the period in {{LEDGER}}, tick Ledger Locked, press **Close period**.                                                                                                                                 |
| Mondays                      | Due dates{{#if SALES_TAX}}, sales tax digest{{/if}} | Look at what is due in the next {{DUE_SOON_BD \| 10}} business days.                                                                                                                                       |

## What each table is for

| Table                            | You use it to                                                                                           |
| -------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Settings                         | The one control row: modes, switches, thresholds, last-run summaries.                                   |
| People                           | Who owns, reviews, approves, and submits.                                                               |
| Periods                          | One row per month. Status shows how far the close is.                                                   |
| Close Task Templates             | The checklist itself. Change the brief first, then this table.                                          |
| Close Tasks                      | This month's checklist. Press **Done** when a task is finished.                                         |
| Documents                        | Statements, exports, and receipts the close is waiting on.                                              |
| Accounts                         | A mirror of {{LEDGER}}'s chart of accounts. Nicknames, never account numbers.                           |
| Reconciliations                  | One per account per month. Unexplained must be zero (or within tolerance) before you press Reviewed.    |
| Entry Schedules                  | Prepaids, depreciation, accruals that repeat. You make a schedule Active; the routine proposes from it. |
| Journal Entries                  | Every prepared entry. Only **Approve for import** lets one into an import file.                         |
| Import Batches                   | Every file handed to you for import, with its version and validation.                                   |
| Tax Obligations                  | The confirmed calendar: what is filed, to whom, when, by whom.                                          |
| Returns                          | One row per filing. Worksheet, due dates, approval, filing, payment.                                    |
| Revenue Sources (Revenue module) | Each system that says what you earned, and how it maps to the ledger.                                   |
| Revenue Recons (Revenue module)  | The monthly bridge from those systems to the ledger.                                                    |
| Questions                        | What the system needs a person to decide. Type the Decision; press **Answered**.                        |
| Run Log                          | Every run, its outcome, and errors.                                                                     |

## The control panel

Open **Control Panel** and edit the Settings row. Never add a second settings row.

| Mode    | What happens                                                                                                                             |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Paused  | The routine writes one log line and stops.                                                                                               |
| Dry Run | It reads and computes, writes its files under `_dry-run/`, and changes nothing in the base.                                              |
| Live    | It creates and updates rows and writes files into the period folders. It still never touches the ledger, the processor, or an authority. |

**Ledger Draft Writes** (off by default) lets Close Prep create approved entries as **drafts** in {{LEDGER}} instead of an import file. You still post them. If it ever finds a journal it created already posted, it turns the switch off and says so in Kill Reason.

**Target Period** and **Target Return** point a routine at one month or one return (for a test or a catch-up month). Clear them afterwards.

If a mode dropped on its own, **Kill Reason** says why. Fix the cause before you raise it again.

## Approving entries

Read **Reason** first: it must say in plain words why the entry exists. Then **Evidence**: the files and record ids behind it. Then **Lines**. **Balanced** and **Accounts Valid** must both be good. Approve with the button. If an entry is wrong, press **Reject** and write why in Agent Notes; if the same mistake repeats, change the policy in the brief, not the entries.

## Importing

1. Open **Imports**. Each batch has a File Path and Validation Notes.
2. Import the file into {{LEDGER}} exactly as the file is. Do not edit it.
3. Save the ledger's import result (or error report) into the same `imports/` folder.
4. Press **Imported**.
5. Export the trial balance, general ledger, and journal list again into `_inbox`.

If some journals failed, the next run writes a `…_FAILED_ONLY_v2.csv` with only those journals, whole. Import that one; never re-import the original, or the journals that succeeded will be duplicated. If a duplicate ever lands, Close Prep drops to Dry Run and tells you which journals to remove in {{LEDGER}}.

## Reconciliations

**Ties** means the statement and the ledger agree after explained items. **Differences** means they do not, and the workpaper lists every unmatched item. A statement line missing from the ledger is recorded by you through the bank feed, never by a journal. Press **Reviewed** only after you read the workpaper. A reviewed reconciliation goes back to Ties or Differences if a new statement or trial balance arrives.

{{#if REVENUE_RECON}}

## Revenue to books

Each month's bridge compares what {{REVENUE_SOURCE_NAMES}} say you earned with what {{LEDGER}} shows, through the clearing accounts. The product and the processor are the record of charges, refunds, fees, credits, waivers, payouts, and amounts owed to others; the ledger is the record of your accounts. Neither is changed to make the other agree.

| Difference class | What it means                                | What you do                              |
| ---------------- | -------------------------------------------- | ---------------------------------------- |
| Timing           | Month-edge records, payouts in transit       | nothing; it reverses next month          |
| Missing entry    | The ledger lacks this month's summary entry  | approve the proposed entry               |
| Mapping          | Booked to the wrong account                  | approve the proposed reclass             |
| Source gap       | The source's own figures do not roll forward | check the export or the key's access     |
| Policy           | The brief does not say how to book it        | answer the question (ask the accountant) |
| Unexplained      | Nothing above explains it                    | answer the question                      |

A waived fee is zero revenue for that record, not a discount. Credits a sponsor pays for are money owed by the sponsor, not a discount.
{{/if}}

{{#if SALES_TAX}}

## Sales tax returns

Each worksheet uses the authority's own line labels. Every line is:

- **VERIFIED**: ties to the ledger and follows a confirmed rate. Enter it.
- **CHECK**: needs your judgement; its question is linked. Settle it first.
- **FILL BY HAND**: only you can enter it (registration numbers, elections, balances shown only in the authority's account, declarations).

A worksheet marked **PRELIMINARY** was prepared before every month was closed. Press **Approve worksheet**, file it yourself through the authority, save the confirmation into the return's folder, press **Mark filed**, pay by the payment due date, and tick **Paid**.
{{/if}}

## Year end

Run `AC · Year-End Package` by hand, one stage at a time: "Run AC · Year-End Package: FY<year>, path Clean, stage E1." Each stage ends with a gate in its report. The tax package's first line says **STOP** if a cross-check fails; do not sign until it is gone. The filing checklist lists every FILL BY HAND item, including identifiers and signatures, which the system never fills. If you ask for filled government forms, open each one in the authority's official reader and check every line against its fill guide.

## Questions

Questions collect in one table. Each has **Why It Matters** and a **Default Treatment**: what the books assume until you answer. Defaults are conservative (no credit claimed, no revenue recognised early, item held in suspense). {{#if ACCOUNTANT}}Questions for {{ACCOUNTANT_NAME}} go out as the questions file each quarter and at year end. You send it.{{/if}}

## Identity rules

| Thing          | Key                                                                   |
| -------------- | --------------------------------------------------------------------- |
| Period         | `YYYY-MM`                                                             |
| Account        | Account Key (the code, or the name when there is none)                |
| Reconciliation | `<Period>\|<Account Key>`                                             |
| Journal entry  | JE Ref (`{{ID_PREFIX}}-J#####`), also in the ledger's reference field |
| Import batch   | Batch ID (`{{ID_PREFIX}}-B####`) and version                          |
| Return         | `<Obligation Code>\|<Period End>`                                     |
| Question       | Question ID (`{{ID_PREFIX}}-Q####`)                                   |

## When something looks wrong

| Symptom                                | Look at                                                                                                                       |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| No digest today                        | The routine's last run in the scheduler, then Run Log → Errors                                                                |
| A file stayed in `_inbox`              | It matched no export type or no account. Rename nothing; check the digest's unrecognised count and tell the agent what it is. |
| Every reconciliation shows Differences | The trial balance export is older than your last import. Export it again.                                                     |
| An approved entry never reached a file | Its Approved By is empty (an API write, not your button), or Accounts Valid is unticked.                                      |
| Import failed                          | Validation Notes, the ledger's import result, and the FAILED_ONLY file                                                        |
| Close Prep dropped to Dry Run          | Kill Reason: usually a duplicate import                                                                                       |
| A due date looks wrong                 | The return's Due Date Source. Computed dates need your confirmation against the authority's account.                          |

## What the system never does

Post or approve a journal. Accept a bank-feed match. Lock a period. File, submit, or pay a return. Refund, transfer, or pay out money. Email anyone. Create a share link. Delete a file or a record. Write an account number, a tax id, or a government id anywhere. Put an amount or a name in a digest.
