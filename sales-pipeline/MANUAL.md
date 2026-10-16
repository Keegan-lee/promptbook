# Sales Pipeline — manual tasks

Steps only a person can do. The build agent gives this list to the person with live links filled in, and ticks off what was done. Never click past an authentication wall on the person's behalf.

## Connections (before the build)

1. **Airtable.** Connect the Airtable MCP or API with access to the workspace where the base will live.
2. **Mail.** Connect the mail connector for the owner's mailbox (read, label, drafts). If teammates' inboxes are ingested, grant the workflow engine access to each one: a service account with domain-wide delegation scoped to **read-only** for ingest, or each person's OAuth.
3. **Calendar.** Connect read-only calendar access for the same people.
4. **Chat.** Connect the chat tool. Note the owner's DM target and the CRM digest channel.
5. **Docs.** Make sure the agent can read the Playbook and Voice guide documents. Keep them free of emoji: some export paths drop characters for each one, which cuts off the END marker.
6. **Optional sources:** meeting recorder, file storage, transactional email provider, and read-only access to the customer system. Use a read-only database role or API key; never an admin key.

## After the base is built

1. **Time zones.** Confirm every date-time field on Outreach Queue and Owed Replies uses `{{TIME_ZONE}}`, 24-hour. The API cannot change a time zone after creation.
2. **Rollup functions.** Open each rollup and confirm the function: Last Activity Date = MAX, Last Human Activity = MAX, Human Activity Count = SUM, the three weight totals on Outreach Settings = SUM, the two "(from Settings)" rollups on Territories = SUM.
3. **Select hygiene.** Look for options with trailing spaces or near-duplicates. Fix them before any data is imported.
4. **Human-activity formula.** Make two `[TEST]` Relationship Activities: one of type Status Change, one linked from a `[TEST]` Outreach Queue row's Sent Activity. `Is Human Activity` must read 0 on both. If it reads 1, fix the formula (`SCHEMA.md` → Activities) in the UI; otherwise the routines' own writes reset Days Since Human Contact. Delete the test records afterwards.
5. **Views.** Create every view listed in `SCHEMA.md` → Views, with its filter and sort.
6. **Interfaces.** Finish the Sales Pipeline interface: Control Panel grids (show Weights Check and Shares Check), Review Queue detail page, Owed Replies with the Draft Link, Hooks, Do Not Contact, Scorecard. Finish the CRM interface. Share them with the people who review.
7. **Automations.** Turn on "Settings change log". Leave "Campaign → Engagement" off unless the brief asked for it.
8. **Settings row.** On the Live row, the owner fills in Signature (reference copy), Mailing Address Line, Playbook URL, and Voice Guide URL, and confirms Mode = Paused.

## Mail plumbing

1. **Mail labels.** Create the labels the routines use: `Sales Pipeline/Outreach`, `Sales Pipeline/Reply Radar`, `Sales Pipeline/Do Not Contact` (or the brief's names). Record their ids if the provider uses ids.
2. **Mailer.** Build or import the mailer workflow. Put its auth secret in the workflow's credential store **and** in the agent runtime's environment. Add the mailer's host to the runtime's network allowlist. Leave the **sending switch off**.
3. **Ingest.** Turn on the mail-ingest workflow after its first supervised run looks right.
4. **Send-as.** If the sender sends from an alias, confirm the alias is verified in the mail provider and set as the mailer's `From`.
5. **Domain health.** If outreach uses its own domain, confirm SPF, DKIM, and DMARC pass, and that the domain is warmed up before Auto-send.

## Routines

1. **Environment.** Create one agent environment for the sales routines with the mailer secret and allowlist. Do not share it with unrelated routines.
2. **Connectors and tool policies.** Open each routine and confirm its connector list and per-tool policies match `KICKOFF.md` Phase 6. A routine created with an empty connector list may have picked up every connector on the account: remove the extras. Reply Radar and the outreach routines must have every send-capable mail tool set to always deny in the policy itself; a sentence in the prompt does not stop a tool call.
3. **Enable in order**, each only after its stage in the kickoff's Phase 7: CRM Daily Reconciliation, then Reply Radar, then each outreach routine.

## The owner's switches

These are the owner's alone. Nobody else moves them, and no routine moves them up.

| Switch                     | Where                   | When                                                                                         |
| -------------------------- | ----------------------- | -------------------------------------------------------------------------------------------- |
| Mode: Paused → Dry Run     | Control Panel, Live row | After the playbook and voice guide v1.0 are approved                                         |
| Mode: Dry Run → Drafts     | Control Panel           | After at least 3 business days of Review Queue rows the owner would send                     |
| Mailer drafting            | Mailer workflow         | With the move to Drafts                                                                      |
| Mode: Drafts → Auto-send   | Control Panel           | After at least 2 weeks of drafts sent with few edits. Set Auto-send Start Date the same day. |
| Mailer sending switch on   | Mailer workflow         | The same day as Auto-send                                                                    |
| Daily Cap, shares, weights | Control Panel           | Any time. The routines read them on every run.                                               |

## Checks a person does once

- `Weights Check` reads "✓ Active weights = 100" and `Shares Check` reads "✓ Shares = 100".
- A Dry Run row appears on Review Queue with a Why Them that cites a real source.
- The smoke-test draft arrived with a clickable, unchanged link and the real signature.
- Owed Replies, Do Not Contact, and Hooks each open from the Sales Pipeline interface.
- An unsubscribe reply, sent from a test address to a test outreach email, ends up on Do Not Contact the same day.
