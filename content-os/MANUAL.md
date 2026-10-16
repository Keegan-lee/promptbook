# Content OS — manual steps

The steps an agent cannot do: they need a person in a tool's UI, a sign-in, or an approval. The build agent hands this list over with a live link on every item, ticks what is done, and records anything deferred in the brief's open items. Do the sections in order.

## 1. Airtable, after the API build

1. **Time zones.** Open every date-time field the pipeline writes (Scheduled DateTime, Sent At, Approved At, Launch At, Published At, Metrics Updated At, Content Mined At, Last Run At, Occurred At) and confirm it displays in `{{TIME_ZONE}}`, with "use the same time zone for all collaborators" on. The API cannot change a field's zone after creation; if one is wrong, create a replacement field and tell the agent its id.
2. **Rollups.** On Content, confirm Posts to Approve and Failed Posts use `SUM(values)` over Needs Approval and Is Failed. The API may not report the rollup function.
3. **Views.** Create every view in `SCHEMA.md`, "Views", with its filter and sort, including the Assets views (Designs to Finish, Ready for Review, Approved Without Media). These stay **unfiltered** forever, because automations watch them:
   - Episodes → Master (long-form module)
   - Clips → Master (long-form module)
4. **Interfaces.** Finish **Content Review** (Review Queue, Post Queue, Asset Review, Calendar, Inventory) and **Content Command Center**. Add the **Approve** button for posts and the **Approve design** button on Asset Review if the stamps use the button design. Share Content Review with every approver and designer in the brief.
5. **Table descriptions.** Paste them from `SCHEMA.md` where the API dropped them.
6. **Email automations.** Open the review email, the error email, and the weekly digest. Confirm each To address against the brief. Airtable sends from its own address; add it to your mail allowlist so it does not land in spam.
7. **Media link fixer** (if your file host needs one). Open its script and confirm it updates **the same record** that triggered it. A fixer that writes into another table fails on every run.
8. **Turn the automations on**, one at a time, after the agent's test record proves each.

## 2. The approval stamp test

The gate depends on this. Do it yourself, signed in as an approver.

1. Open a `[TEST]` post. Tick Copy Approved and Creative Approved, then set Approval to Approved (or click Approve). Approved By becomes you, and Approved At is now.
2. Ask the agent to set Approval to Approved on a second `[TEST]` post through the API. Approved By must stay **empty**.
3. Set the first post to Changes Requested. Both stamps clear and both boxes untick.
4. Assets: open a `[TEST]` asset in Asset Review and approve it (button, or Status = Approved). Approved By becomes you. Have the agent set a second `[TEST]` asset to Approved through the API: Approved By stays empty, and the post it is linked to keeps Creative Ready = 0.
5. If step 2 or step 4 stamped anything through the API, stop. The agent fixes the automation (`SCHEMA.md`, "Approval stamps: how to build them"). Do not go on with a gate that an API write can open.
6. Delete the test posts and assets, and tell the agent.

## 3. Distribution service

1. Create the profiles the brief names (one for the brand, one per personal account) in `{{DISTRIBUTOR | Upload-Post}}`. The agent reads their exact names afterwards; it never types one from memory.
2. Connect each social account to its profile. This is an OAuth sign-in on the platform, so the account's owner does it, in the distributor's dashboard or through a connect link the agent asks for and hands over (the link is valid for a limited time; on an account with no profile it also creates one). A personal account is connected by its owner, never by someone else.
3. Create an API key and add it to the workflow engine's credential store (section 4), or connect the distributor's connector to the agent runtime if distribution runs as a routine. Never paste a key into chat.
4. Tell the agent. It reads the profiles and connected platforms, and the page, organisation, and location ids, and checks them against Platform Rules.
5. A channel that is not connected yet has Delivery Method = Manual in Platform Rules (or Is Active off, if nothing should be drafted for it yet). When a platform later needs reconnecting, Distribute stops sending to it and writes one error row a day until its owner reconnects it.
6. X: if the account has no link add-on, the distributor strips links from X posts and first comments. Write "no links: they are stripped" into the X channel's Caption Style Notes.
7. **No workflow engine:** the agent builds `CP · Distribute` as a routine. Set its always-deny tools from `workflows/distribute.md` as tool policies, and update its schedule whenever a channel's Posting Time (Local) changes.

## 4. Workflow engine

1. Create the credentials, named as the agent asks:
   - Airtable: a token scoped to the Content OS base only. Prefer a token owned by a separate integration account that is in no Allowed Approvers list (see the approval stamp design).
   - The distributor's API key, as a header credential.
   - The file host: an OAuth2 credential with the sharing scope (Dropbox: `sharing.write`), for the media-links job.
   - The CMS: a **read-only** token for the inventory sync.
   - Design exports to the file host (creative module, without G.7): none; a person exports.
2. Set `CP · Error Handler` as the error workflow on every `CP ·` workflow, and each workflow's time zone to `{{TIME_ZONE}}`.
3. Activate workflows only when the go-live ladder reaches them (kickoff, Phase 7), in this order: Error Handler, CMS Inventory Sync, Scheduler, Distribute and Confirm Posts, Analytics Pull.

## 5. Agent routines

1. Create the routine environment the agent names. Add the secrets it asks for in the environment's secret settings.
2. **Network allowlist:** add the file host's direct-download domain (Dropbox: `*.dl.dropboxusercontent.com`) for the Meeting Idea Miner. The sandbox blocks it otherwise. For Creative Drafts, add the design tool's export download domain, and, if it saves final files itself, the file host's API and upload domains.
3. **Creative Drafts file token** (only if the brief lets it save final files): create a file-host app limited to the media folder (Dropbox: an app-folder app with `files.content.write`, `files.metadata.read`, and `sharing.write`), and put its token in the routine environment's secrets under the name the agent gives you.
4. Connect only the connectors each routine lists, and set its "always deny" tools.
5. Read each routine's Dry Run output with the agent before you enable it. Enable one routine at a time.

## 6. Brand pack and design tool

1. Read brand pack v1.0 end to end and approve it. No routine that writes copy runs before that.
2. Check the design tool's plan has brand kits and brand templates (Canva: a team plan). Connect the design tool's connector **signed in as an account in the brand's team**, never a personal workspace: the creative routine checks the brand kit is visible and stops otherwise.
3. Set up the brand kit (`BRAND_PACK_TEMPLATE.md`, last section).
4. Make or confirm one brand template per channel and kind in the brief's template map. Put the placeholder texts the Field Map names into each (for example `HEADLINE TEXT`), or tag data fields where the plan supports autofill. Open each template once and check it.
5. Create a drafts folder for the routine's designs, if the brief names one.
6. Tell the agent. It reads the templates' ids and fields, fills the Templates table, and runs the creative rung of the go-live ladder.

## 7. Designs (the creative module, every week)

1. A new draft design arrives as a Draft asset in Content Review → Asset Review, with a preview and an edit link. The copy is already placed, or listed in a comment.
2. Open it in the design tool, finish it (images, layout, anything the template could not do), then set the asset to Ready for Review.
3. An approver checks it against the brand pack and approves it with the **Approve design** button (or Status = Approved), signed in as themselves. Only that stamp lets the post go out.
4. Changes after approval: set the asset back to In Progress first. A design changed after its approval is never exported until it is approved again.
5. Want a fresh draft? Set the post to Changes Requested with Review Notes starting `CREATIVE: redraft`. Any other `CREATIVE:` note is passed to the design as a comment.
6. Video and voice: an editor's agent may rough-cut, caption, or draft a read. A person makes the final cut and approves every read before its asset is approved.

## 8. Recorder and meetings (meetings module)

1. Decide who ticks **Use for Content** on the opt-in log, and when. Only a person ticks it.
2. Tell everyone who joins recorded meetings that opted-in meetings feed marketing ideas, that names, money, and anything said in confidence are removed, and how to say "off the record".
3. Confirm the recorder's coverage start date for the brief.

## 9. CMS (CMS module)

1. Create the read-only token for the inventory sync.
2. Article routine only (no blog kit): a token that can create drafts and upload assets, and nothing more. Confirm the test dataset exists.
3. Publishing and setting a document live stay with a person. "8 · Set live in <your CMS>" in Next Manual Step means that switch is still off.

## 10. Voice (only if a cloned voice is used)

1. The voice owner gives written consent. Record where it is kept in the brief.
2. The voice owner creates the clone in the voice tool themselves.
3. Every generated read is listened to and approved by a person before its asset is approved. A clone is marked Synthetic Media on its asset.

## Checks

- Next Manual Step is blank on an Archived row, reads `9 · Held (<series>)` on a held row that is not Published, and reads `5 · Finish and approve designs` on a row whose post has a Draft asset.
- Content Review → Review Queue shows the rows in Needs Review; Post Queue shows the posts.
- A `[TEST]` idea promoted twice creates one Content row.
- An idea with Sensitivity Flag ticked does not promote until Redaction Status is Human-Checked.
- One throwaway Pipeline Errors row sends one email. Mark it Ignored.
- The Command Center numbers match the views they summarise.
