# Blog writing — manual steps

The steps only a person can do. The build agent replaces each `<link>` with a live link from the build, ticks what it could do itself, and hands the rest over. Do not let an agent click past a sign-in wall for you.

## After the Airtable build

1. **Views.** Create every view in `SCHEMA.md` ("Views") with its filter and sort: <link to Blog Backlog>, <link to Keyword Ledger>, <link to Run Log>.
   - Blog Backlog → **All** stays unfiltered. Automations watch it. Do not filter or delete it.
   - **Live Posts** shows exactly the fields the schema lists. Other kits read it.
2. **Select options.** Open each select and look for trailing spaces or typos. If you rename an option, rename it in the brief and the ID map too, then tell the agent so it can refill the routines.
3. **Time zones.** Confirm every date-time field shows `{{TIME_ZONE}}`. The API cannot change a date-time field's zone after it is created.
4. **Interface: Blog Desk** (<link>). Finish the pages the agent drafted:
   - Triage: record review on the Triage view. Add buttons or editable fields for Status (Ready, Rejected, Parked), Priority, and Review Notes.
   - Review: Drafts to Review, with Draft URL and Preview Path as clickable links, and Status editable to Changes Requested or Live.
   - Pipeline, Blocked, Keywords, Runs: as the schema lists.
   - Settings: the Blog Settings record. Make it editable for the owner only.
   - Share Blog Desk with the topic approvers and reviewers in the brief. The owner does the sharing.
5. **Automations** (<link>). Turn each on only after a test proves it, then delete the test record:
   - **Topic approval stamp.** Set a test ticket to Ready in the UI as yourself: Approved By becomes you. Then have the agent set another test ticket to Ready through the API: Approved By must stay empty. If the API write stamps, the automation is wrong; fix it before going on.
   - **Clear stale stamp.** Move the test ticket back to Proposed: both stamp fields clear.
   - **Live stamp.** Set a test ticket to Live: Live At fills.
   - **Error email.** Create a Run Log row with Outcome Error: the owner gets one email with the run, the routine, and the one-line error.
   - **Draft ready email** (only if there is no chat connector): set a test ticket to Draft Ready: the reviewer gets one email.
6. **Existing-task-table model only.** Confirm the `blog`, `blog-geo`, `blog-depth`, `blog-refresh`, `blog-news`, `blog-correction` tags (and the brief's own kinds) exist on the task table, and that no automation on that table reacts badly to tickets created by an agent.

## Connect the keyword tool

1. Connect the tool in the agent platform's connector settings, signed in as the account that holds the plan (Ubersuggest by default). Never paste a key into the chat.
2. For API-key tools (Semrush, Ahrefs, DataForSEO), put the key in the routine environment's secret store. Google Keyword Planner needs a Google Ads developer token and an OAuth client: a person sets those up in Google's consoles.
3. Where the platform supports per-tool policies, set every forbidden endpoint from `KEYWORD_TOOL.md` to **always deny** on every routine (for Ubersuggest: `generate_article`, `keyword_metrics`, `article_title_suggestions`, `get_article`, and the project, audit, backlink, and brand tools).
4. If a cloud routine cannot reach the tool, add its API domain to the environment's network allowlist.

## CMS token scopes

Give each routine the narrowest token that does its job. The Gap Scan and the Refresh Review only read.

| CMS        | Writer token                                                                                                                                                                                                                                        | Read-only token (scan, refresh)    |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| Sanity     | A robot token on the production and test datasets. If your plan offers a role that can create and edit drafts but not publish, use it. Otherwise an Editor token, with publish, unpublish, discard, and schema tools denied by tool policy.         | A Viewer token                     |
| WordPress  | An application password for a user with the **Contributor** role: drafts yes, publish no. Contributors cannot upload media, so covers stay with a person; an Author can upload but can also publish, so weigh it.                                   | Public REST reads, or a Subscriber |
| Webflow    | A site token with CMS read and write only. No site-publishing scope. The writer never calls a publish endpoint.                                                                                                                                     | A site token with CMS read only    |
| Ghost      | If your version offers staff access tokens, a **Contributor** staff user's token: drafts yes, publish no. An integration's Admin API key cannot be limited below admin; if you must use one, keep it in the secret store and rely on the deny list. | The Content API key                |
| Contentful | A personal access token for a user whose role can create and edit entries and upload assets but not publish (a custom role). Otherwise an Editor token with publish denied by tool policy.                                                          | A Content Delivery API token       |

- Supporting documents (only if the brief turns them on) need the writer's token to create published documents of those types. A draft-only role cannot; then leave the option off.
- The preview secret stays in the site's environment. It never goes into Airtable, a routine prompt, or a chat message.
- If the cloud sandbox blocks the CMS's asset CDN, add that domain to the network allowlist.

## The publish step (always a person)

The engine stops at a draft. This is the part that stays human:

1. Open the draft from Blog Desk → Review (Draft URL).
2. Read it against the ticket's ACCEPTANCE list. Check every number against its source.
3. Edit it in the CMS, or set the ticket to Changes Requested with Review Notes. Once you edit it in the CMS, finish it there: the writer will not overwrite your edits.
4. Set the publish date. Confirm the category, author, reviewer, and review date. Add a cover if Agent Notes says `needs cover`.
5. Publish.
6. Open the preview (the Preview Path with your preview secret, or the CMS's own preview) and check it reads right.
7. Switch the live flag on. **Published alone is not public.** Do the same for every supporting document the reviewer message lists under "also switch live" (a glossary term, a location page): each has its own switch.
8. Nothing to do in Airtable: the writer's next run sets the ticket to Live. You may set it yourself.

For a refresh, set the review date to today when you publish. For a correction, check the dated entry shows in the sources and corrections block.

## Routines

1. Enable each routine only after its stage in Phase 6 of the kickoff has passed, in this order: Gap Scan, Post Writer, Refresh Review.
2. Set the deny lists from each template's Install table as tool policies.
3. **Desktop scheduled task** (if the writer runs on a desktop): install the design skill or image app on that machine, keep it signed in, and set its power settings so it is awake at the scheduled times. A machine that was asleep fires missed runs late; Blog Settings → Writer Min Gap Hours stops a second run from drafting the same day.
4. **Embed builder** (only if the brief lets the writer build embeds): connect the product's connector signed in as the one account the brief names. A replaced key or token usually takes effect only in a new session: after a swap, start a fresh routine run and check the writer's first `whoami`-style line in the Run Log.
5. Move each mode on Blog Settings yourself: `Paused → Dry Run → Live`. An agent never raises a mode. Set Writer Min Gap Hours back to the brief's value when verification ends.

## Checks

- Next Manual Step reads `4 · Approve or reject the topic` on a Proposed ticket and is blank on a Live one.
- An API write of Status = Ready leaves Approved By empty; your UI change fills it.
- Run Log → Errors is empty after verification.
- No `[TEST]` record is left in the base, and the test documents are discarded or unpublished in the test dataset.
