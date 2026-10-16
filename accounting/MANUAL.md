# Accounting — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past a sign-in wall on the person's behalf, and never type a password, an account number, or a tax id for them.

## Connections (before the build)

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the base will live.
2. **File storage.** Connect the file host for the accounting root, or choose a local folder and a desktop runtime. The agent needs read and write inside that one folder only.
3. **Ledger.** If the ledger has a connector, sign in through the platform's connector settings with a **read-only** role where the ledger offers one (an accountant or reporting role, not an admin). If it has no connector, nothing to do here: exports go into `_inbox`.
4. **Payment processor** (Revenue module). Create a **restricted, read-only** key in the processor's dashboard (read access to balance, charges, refunds, disputes, payouts, fees; no write scopes). Add it to the routine environment's secret store, never to chat.
5. **Product database** (Revenue module). Create a database role that can `SELECT` only the tables the bucket map names, and nothing else. Put its connection string in the secret store.
6. **Chat.** Connect the chat tool. Note the private finance channel or DM the digests go to. Never a public channel.

## After the base is built

1. **Time zones.** Confirm every date-time field shows `{{TIME_ZONE}}`, 24-hour. The API cannot change a date-time field's zone after it is created.
2. **Currency fields.** Confirm each shows `{{BASE_CURRENCY}}` with the right number of decimals.
3. **Rollups.** Open each rollup and confirm the function is SUM: Periods → Tasks Open, Recons Not Tied, Entries Pending, Documents Missing, Blocking Questions.
4. **Select hygiene.** Look for trailing spaces and near-duplicates in every select. Fix them before data arrives.
5. **Views.** Create every view in `SCHEMA.md` → Views, with its filter and sort. Periods → All and Journal Entries → All stay unfiltered: do not filter or delete them.
6. **Interface: Close Desk** (<link>). Finish the pages the agent drafted and add the buttons:
   - Entries: **Approve for import** and **Reject**.
   - Reconciliations and Revenue: **Reviewed**.
   - Imports: **Imported**.
   - This Close: **Done**.
   - Questions: editable Decision and **Answered**.
   - Returns: **Approve worksheet**, **Mark filed**, editable Confirmation File and Paid.
   - Period: **Close period**, shown only when Ledger Locked is ticked.
   - Control Panel: the Settings record, editable by the owner only.
     Share Close Desk with the finance users the brief names. The owner does the sharing.
7. **Accountant View** (optional). Finish and share it with the accountant only if the owner wants to.
8. **Automations** (<link>). Turn each one on only after a test proves it, then delete the test records:
   - For each button: click it on a `[TEST]` record in the interface and see your own stamp. Then have the agent set the same status through the API on another `[TEST]` record: the stamp must stay empty, and the next routine run must report it as "unstamped approval". If the API write stamps, the automation is wrong; fix it before going on.
   - **Close period:** with Ledger Locked unticked, the button must refuse.
   - **Settings change log:** change a Mode on the Settings row: the owner gets one email.
   - **Run error email:** create a Run Log row with Outcome Error: the owner gets one email.
9. **Settings row.** The owner checks the mechanics, confirms every Mode is Paused and Ledger Draft Writes is off.

## Ledger set-up

1. **Import template.** Download the ledger's own sample journal import file and save it in `{{ACCOUNTING_ROOT}}/templates/`. Tell the agent where it is.
2. **Accounts the kit needs.** Create in the ledger any account the agent lists as Proposed (clearing accounts per processor and currency, a suspense account, one owner-balance account per person). The agent never creates accounts in the ledger. Then export the chart of accounts again.
3. **Test import.** Import the agent's one-journal `[TEST]` file into the ledger's sandbox company, or into the live ledger as a draft, and delete it afterwards. Save the ledger's import result next to the file.
4. **Lock dates.** Learn where the ledger sets its lock or closing date. You set it at every close (task C60). The agent never does.
5. **Bank feeds.** Keep categorising bank and card feeds yourself in the ledger. The agent lists what is missing; it does not accept matches.

## Folders

1. Confirm the accounting root and its subfolders exist, and that sharing is limited to the finance users.
2. Each month, put statements and exports into `_inbox` (any name). The routine files them.
3. After every import, export the trial balance, the general ledger, and the journal list again into `_inbox`.
4. Never delete anything under `_inbox/_filed/` or in a period folder. Versions are how the record stays honest.

## Routines

1. **Environment.** One environment for the accounting routines, with the processor key and database role (if any) as secrets and the file host's download domain on the network allowlist if the sandbox blocks it. Do not share it with unrelated routines.
2. **Tool policies.** Confirm each routine's deny list matches KICKOFF Phase 6. No routine has an email send tool, a ledger posting tool, a processor money-moving tool, or a file delete tool.
3. **Enable in order**, each only after its step in KICKOFF Phase 7 is clean: Close Prep, Revenue Reconciliation, Sales Tax Prep. Year-End stays manual.

## The owner's switches

These are the owner's alone. No routine moves them up.

| Switch                             | Where         | When                                                                                   |
| ---------------------------------- | ------------- | -------------------------------------------------------------------------------------- |
| Close Prep Mode: Dry Run           | Control Panel | For the known-good-month test                                                          |
| Close Prep Mode: Live              | Control Panel | After the dry run on a closed month is clean                                           |
| Revenue Recon Mode: Dry Run, Live  | Control Panel | Same, on the same month                                                                |
| Sales Tax Mode: Dry Run, Live      | Control Panel | After a worksheet matches a return you already filed                                   |
| Year-End Mode                      | Control Panel | Before running the year-end package                                                    |
| Ledger Draft Writes                | Control Panel | Only if the ledger keeps drafts unposted, and after three months of clean import files |
| Tolerances, thresholds, lead times | Control Panel | Any time. Change the brief first, then the row.                                        |
| Target Period, Target Return       | Control Panel | For tests and catch-up months. Clear them afterwards.                                  |

## What only a person ever does

- Approve or reject an entry. Import a batch and save the import result. Post entries in the ledger.
- Record bank and card transactions through the feeds.
- Lock a period in the ledger and press Close period.
- Approve a worksheet, file a return, pay a tax, record the confirmation.
- Send anything to the accountant, an authority, a vendor, or a customer.
- Pay a vendor, a contractor, an owner, or a third party their payout.
- Enter a registration number, an account number, a government id, or a signature.

## Checks a person does once

- The dry-run close on a known-good month: every reconciliation tied, and every proposed entry was either zero or a real finding.
- A `[TEST]` approval through the API produced no stamp and the routine ignored it.
- The test import landed, and Close Prep found its JE Ref in the next journal export.
- A sales tax worksheet for a filed return matched what was filed, line by line.
- No digest so far has carried an amount, a name, or an account number.
