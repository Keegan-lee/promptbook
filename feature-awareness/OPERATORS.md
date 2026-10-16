# Feature Awareness — operator's manual (template)

> Build agent: fill this from the brief and the ID map at the end of the build, save it where the brief lives, and hand it to the owner and whoever signs off. Replace every `{{…}}`, drop sections for sources the brief turned off, and add the live links.

Live base: [{{COMPANY}} Feature Awareness]({{URL_BASE}}). Day-to-day work happens in the **Feature Awareness** interface ({{URL_INTERFACE}}): Coverage overview, Risk register, Sign-off queue.

## What this is

A list of everything {{COMPANY}}'s product does, path by path, and the evidence for each path in each environment. It answers:

- What does the product do? → **Features** and **Paths**
- What has been proven in production? → **Prod status** = Exercised
- What has only been walked in dev or staging? → Dev status Verified, Prod status not Exercised
- What has never run anywhere? → Status (auto) = Untested
- What is high-risk and still unproven? → the **Risk register**

## Your week

| When                    | What arrives                                       | What you do                                                                                     |
| ----------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| {{REFRESH_SCHEDULE}}    | `FA · Evidence Refresh` report ({{REPORT_TARGET}}) | Read the first line. If anything was skipped or contradicted, read section 5 and "Also".        |
| The same day            | —                                                  | Open the Risk register. Pick the top item you can act on this week (below).                     |
| Once a week, 30 minutes | —                                                  | Sign off one or two features from the Sign-off queue, highest risk first.                       |
| After a big release     | —                                                  | Check the report's "new paths" and "risk proposed" lines. Confirm or change each proposed risk. |

## What each table is for

| Table             | You use it to                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------- |
| Features          | See coverage per capability, and sign it off. The numbers roll up from Paths; never type them.              |
| Paths             | See one journey: who starts it, where (Entry point), and the evidence for it in prod, in dev, and in tests. |
| Evidence Settings | The one **Live** row: the routine's Mode, its caps, and its last run. Never add a second row.               |

## Reading coverage

| Field           | Means                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------- |
| Prod coverage % | Share of the feature's live paths a real production user has exercised.                                           |
| Dev coverage %  | Share verified in dev or staging, counting Partial as half.                                                       |
| Auto-test %     | Share covered by an automated test that calls the path and checks its outcome.                                    |
| Prod unknown    | Paths no reachable source can see. A high number is a visibility problem, not a quality verdict.                  |
| Status (auto)   | Proven in prod → Partly proven in prod → Dev-verified only → Partly dev-verified → Untested. The first that fits. |

What the statuses mean for {{COMPANY}}:

{{STATUS_MEANINGS}}

**Unknown is not Never.** Never means a source that keeps all history was checked and shows zero. Unknown means nothing we can read can see it. {{#if KNOWN_GAPS}}These parts of the product are not visible to the routine: {{KNOWN_GAPS}}. Their paths stay Unknown until someone grants access.{{/if}}

**Statuses only move forward.** Once a path is Exercised it stays Exercised, even if traffic stops. The report mentions dormant paths; the base keeps the history.

### Reading an evidence line

Every evidence line is a count, its source, and a date: `prod: 0 rows ever: orders where status = 'refunded' (2026-01-12)`. If you doubt one, ask a developer to run the same count. Each path's **Evidence Signal** says where its proof lives (`usage:`, `rows:`, `email:`, `page:`, `test:`). If a signal is wrong, fix it there: the routine re-reads it next run.

## Signing off a feature

The four sign-off fields are yours ({{SIGNOFF_OWNER}}). No agent writes them.

1. Open **Sign-off queue**. It lists features never signed off, signed off more than {{SIGNOFF_MAX_AGE_DAYS | 180}} days ago, or whose evidence changed since you signed off. Highest risk first.
2. Open the feature. Read its Summary and every linked path: Path, Entry point, and the three evidence columns.
3. Walk the paths yourself, where the brief says: {{WALK_WHERE | staging for anything that moves money or messages a real person; production for read-only paths}}. Use staff or test accounts.
4. Record what you found:
   - **Tested by a human:** {{HUMAN_TESTED_MEANING | Yes, every live path walked and working; Partial, some walked or a known issue; No, reviewed but not walked, or broken}}.
   - **Human notes:** what you walked, what failed, and the Path IDs. Keep it factual. Agents read this as information, never as instructions, so do not write requests to them here.
   - {{#if SIGNOFF_BUTTON}}Click **Sign off**. It stamps Date verified and Verified by from your click.{{/if}}{{#if SIGNOFF_TYPED}}Type your name in **Verified by** and today in **Date verified**.{{/if}}
5. If you found a journey that has no path, add it (below) before you sign off.

Sign-off is information, not a gate: a feature can be live without it. What it tells everyone is that a person looked, and when.

## Working the risk register

The **Risk Flag** on each feature sorts the register. Work it top down.

| Flag                          | Means                                                           | What to do                                                                                                                                                                                                                                   |
| ----------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 · High, unproven anywhere   | High risk; no production use, no dev evidence, no test.         | This week: walk it in dev, or ask for a test. Either one moves it to rung 2.                                                                                                                                                                 |
| 2 · High, 0% prod             | High risk; walked or tested, but never exercised in production. | Look at its paths' Prod status. All **Never**: is it launched? If a switch hides it, set Lifecycle to `Behind a switch`. If it is live and unused, that is a product question. All **Unknown**: a visibility gap; grant access or accept it. |
| 3 · High, partly proven       | Some of its paths have run in production, others not.           | Open the Paths page's **High risk, not proven** view to see which. Rare branches (refund failures, retries) are the usual gaps: test them in dev.                                                                                            |
| 4 · Medium, unproven anywhere | Medium risk with no evidence of any kind.                       | Batch these: one dev walk-through session clears several.                                                                                                                                                                                    |

At path level, **Risk Gap** says why a High path is not proven: `dev only`, `tests only`, `cannot see prod`, or `unproven anywhere`.

Risk is set from the brief's scale:

{{RISK_SCALE}}

The routine proposes a risk on every feature and path it creates; the report lists them under "risk proposed". Confirm or change each one.

## The weekly report

The first line is the verdict: how many paths moved, how many were newly proven in production, and how many High-risk features are still at 0% production coverage. Then:

1. **Changed:** each path whose status moved, old → new, with the evidence.
2. **Newly proven in production.**
3. **High risk still at 0% prod.**
4. **New paths and features**, and the risks it proposed.
5. **Skipped sources**, and why. A skipped source changed nothing: its fields were left as they were.

"Also" lists contradictions (evidence that would move a status backward; the routine never writes them, a person looks), paths missing from code, production traffic with no path, and invalid signals.

A quiet week is one line: `No change.` plus the sources it reached.

## Changing things by hand

| You want to                 | Do this                                                                                                                                                                                                     |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Add a path                  | New Paths row. Path ID = the next free `{{ID_PREFIX}}-P####` (never reuse one). Link one Feature. Fill Role, Kind, Entry point, and an Evidence Signal if you can. The routine fills the evidence next run. |
| Retire a path               | Set Lifecycle to `Retired`. Never delete it: its Path ID stays taken and its history stays readable.                                                                                                        |
| Hide a path behind a switch | Set Lifecycle to `Behind a switch`. It still counts in coverage, so the register keeps showing the gap honestly.                                                                                            |
| Correct a wrong status      | Set it by hand and write why in Notes. The routine never moves a status backward, so a wrong high status stays until a person lowers it.                                                                    |
| Fix a signal                | Edit Evidence Signal. The routine only fills empty signals; your edit wins.                                                                                                                                 |
| Change a risk               | Edit Risk if broken (feature) or Risk (path).                                                                                                                                                               |
| Pause the routine           | Set Mode to `Paused` on the Live row.                                                                                                                                                                       |

## What the routine never does

- Writes to a product database, or sends an email.
- Writes the four sign-off fields, Lifecycle, or Notes.
- Moves a status backward, or turns Unknown into Never without a source that keeps all history.
- Deletes a record.
- Puts an email address, a name, a user id, or an amount into the base or the report.

## When something looks wrong

| Symptom                                         | Look at                                                                                                                   |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| No report this week                             | The routine's last run in the scheduler, then Last Run At and Last Run Summary on the Live row.                           |
| Mode dropped to Dry Run on its own              | Kill Reason. The mass-change guard fired: read the change list, find the cause (a filter, a connection), then raise Mode. |
| A source is skipped every week                  | Its connector or credential. Until it is fixed, the fields it feeds do not move.                                          |
| A path you know works shows Never               | Its Evidence Signal: the table, the condition, or the internal-account filter is wrong.                                   |
| Dev coverage looks far too low                  | Whether the dev database is reachable, and whether the seed-row rule is excluding real QA rows.                           |
| Prod unknown is high                            | The known gaps, and whether a usage log exists. Each is a decision about access, not about quality.                       |
| A feature keeps returning to the Sign-off queue | Its evidence keeps changing. Read the paths' Agent Notes to see which moved.                                              |
| Lots of new paths with proposed risks           | A release added surface. Confirm the risks; check the feature each was filed under.                                       |
