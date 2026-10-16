# Content OS — schema

The field contract for the Content OS base: one row per piece of content, one row per channel post, one row per channel's rules. Build it with [`../_shared/AIRTABLE.md`](../_shared/AIRTABLE.md). The kickoff ([`KICKOFF.md`](KICKOFF.md)) decides which modules are on and fills the vocabularies. Routines and workflows address every field by id, through the brief's ID map.

Values in `{{…}}` come from the brief. Everything else is a mechanic. Keep mechanics exactly as written: the formulas, routines, and workflows match on these strings, and a renamed option silently breaks a filter.

Base name: `{{BASE_NAME | <Company> Content OS}}`.

## Modules

| Module           | Tables                                                                    | On when (interview answer)                                            |
| ---------------- | ------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Core             | Settings, Content, Ideas, Platform Posts, Platform Rules, Pipeline Errors | Always                                                                |
| Core (media)     | Assets                                                                    | Any post carries media, or distribution is on                         |
| Meetings         | Source Recordings                                                         | Block E: meetings are an idea source                                  |
| Long-form series | Episodes, Clips                                                           | Block G: a podcast, video show, or webinar series                     |
| Angles           | Content Angles                                                            | Block G: you want hook options per piece [on]                         |
| Scripts          | Scripts                                                                   | Block G: you film scripted video or record voiceover                  |
| Articles         | Articles                                                                  | Block H: long-form articles drafted outside the CMS                   |
| Newsletter       | Newsletter Issues                                                         | Block D: a newsletter is a content type                               |
| Creative         | Templates, plus the `CP · Creative Drafts` routine                        | Block G: a design tool connector with a brand kit and brand templates |

A module that is off has no table, no links to it, no lookups from it, and no lines in any formula that reads it. Each formula below says which lines to remove.

## Build order

1. Settings and Platform Rules (no links)
2. Ideas
3. Content (scalar fields, and formulas that need no links)
4. Assets
5. Module tables: Source Recordings, Episodes, Clips, Articles, Content Angles, Scripts, Newsletter Issues, Templates
6. Platform Posts
7. Pipeline Errors
8. Links in pairs, then lookups, then rollups, then the formulas that read them. Next Manual Step is last: it reads two rollups, which read two formulas on Platform Posts.

## Time

- Every date-time the pipeline writes displays in `{{TIME_ZONE}}`. Set the zone when you create the field. The API cannot change it later.
- The API reads and writes date-times in UTC whatever the display zone says. Convert before you compare.
- Posting times are a single-line `HH:mm` field named **Posting Time (Local)**. Do not add a UTC date-time "default posting time" next to it. The system this kit came from kept one beside the local field, and it was the wrong one to read.

## Shared vocabularies

Use these exact strings on every table that has the field.

| Vocabulary   | Fields                                                                                   | Values                                                                                                                                                                                                                           |
| ------------ | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Content type | Content.Type, Ideas.Suggested Type                                                       | `{{CONTENT_TYPES}}` from Block D. Default: Text Post, Single Image, Carousel, Promo Video, Clip, Podcast Episode, Script, Blog Post, Glossary Term, Newsletter. Keep the spelling of any default you use: formulas branch on it. |
| Platform     | Ideas, Content, Scripts → Target Platforms; Platform Posts and Platform Rules → Platform | `{{PLATFORMS}}` from Block A. Brand channels use the platform's own name (`LinkedIn`, `Instagram`). A personal account is `<Platform> — <Person> (personal)`, so nobody mistakes it for the brand page.                          |
| Series       | Content.Series                                                                           | `{{SERIES}}` from Block D                                                                                                                                                                                                        |
| Topic        | Ideas.Topic, Episodes.Themes                                                             | `{{TOPICS}}` from Block D                                                                                                                                                                                                        |
| Aspect ratio | Content, Assets, Platform Rules                                                          | 9:16, 16:9, 1:1, 4:5                                                                                                                                                                                                             |
| Content mode | Platform Posts.Content Mode, Platform Rules.Supported Content Modes                      | Video post, Text-only, Image/Carousel, Link post, Document/PDF Carousel, Thread                                                                                                                                                  |
| Weekday      | Platform Rules.Posting Days                                                              | Mon, Tue, Wed, Thu, Fri, Sat, Sun                                                                                                                                                                                                |

Every option on Platform Posts → Platform must also exist on Platform Rules → Platform, spelled the same. The Copy Writer reads a channel from Platform Rules and writes it onto a post. It never typecasts, so a missing option is an error, not a new misspelled option.

## Ids

| Table          | Formula                                                   |
| -------------- | --------------------------------------------------------- |
| Content        | `"{{ID_PREFIX}}-C" & RIGHT("0000" & {Content Number}, 4)` |
| Platform Posts | `"{{ID_PREFIX}}-P" & RIGHT("0000" & {Post Number}, 4)`    |
| Ideas          | `"{{ID_PREFIX}}-I" & RIGHT("0000" & {Idea Number}, 4)`    |
| Episodes       | `"{{ID_PREFIX}}-E" & RIGHT("000" & {Episode Number}, 3)`  |
| Clips          | `{Episode} & "-C" & RIGHT("00" & {Clip Number}, 2)`       |

People search these ids. They cannot search Airtable record ids. Every job report and notification names rows by these ids.

## Settings

One row per routine and workflow. Every job reads its own row before it does anything. Primary **Job**.

| Field             | Type          | Notes                                                                                                     |
| ----------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| Job               | single line   | Exact job name, for example `CP · Copy Writer`.                                                           |
| Kind              | single select | Agent Routine, Workflow                                                                                   |
| Mode              | single select | Paused, Dry Run, Drafts, Live. Only the owner raises it. A guard may lower it and must write Kill Reason. |
| Kill Reason       | long text     | Dated line from the guard that lowered Mode.                                                              |
| Last Run At       | date-time     |                                                                                                           |
| Last Run Summary  | long text     | Counts and record ids only. Newest line on top. Keep the last 20 lines.                                   |
| Guidance Versions | single line   | Versions of the binding documents the job last read, for example `brand v1.2`.                            |
| Notes             | long text     |                                                                                                           |

Seed one row per job the brief turns on, Mode = Paused. What each mode means:

| Job                                         | Dry Run                                                                    | Drafts                                                             | Live                                                                                                  |
| ------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| CP · Copy Writer                            | Plans in Last Run Summary. Creates nothing.                                | Same as Live                                                       | Creates posts at Needs Review. Never approves.                                                        |
| CP · Meeting Idea Miner                     | Selects, redacts, leak-checks. Creates no Ideas, stamps no log rows.       | Same as Live                                                       | Creates Ideas at Triage.                                                                              |
| CP · Article → CMS Draft                    | Builds and checks the document. One `DRY RUN` line in Article Notes.       | Same as Live                                                       | Creates CMS drafts. Never publishes.                                                                  |
| CP · Creative Drafts                        | Plans in Last Run Summary: post, template, field lengths. Creates nothing. | Same as Live                                                       | Creates draft designs and Draft assets. Never approves.                                               |
| CP · Scheduler                              | One `DRY RUN` line per post with the slot it would set.                    | Same as Dry Run                                                    | Fills Scheduled DateTime.                                                                             |
| CP · Distribute                             | One `DRY RUN` line per post it would send.                                 | Same as Dry Run: a distributor has no draft state on every channel | Sends through the gate. As an agent routine, it also runs Confirm Posts' steps under that row's Mode. |
| CP · Analytics Pull                         | Writes metrics and scores. Lists back-to-review candidates, moves none.    | Same as Dry Run                                                    | Also moves back-to-review posts.                                                                      |
| CP · Confirm Posts, CP · CMS Inventory Sync | Paused or Live only. They read a source and write Airtable.                |                                                                    |                                                                                                       |
| CP · Error Handler                          | Ignores Mode. It always runs.                                              |                                                                    |                                                                                                       |

## Content

One row per piece. Primary **Title**. Table description: "Source of truth for everything we make and publish."

| Field                                                                 | Type                  | Notes                                                                                                                       |
| --------------------------------------------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Title                                                                 | single line           | A title starting `[TEST] ` marks a test record. Every job that sends or publishes routes it to a test target or skips it.   |
| Content Number                                                        | autonumber            |                                                                                                                             |
| Content ID                                                            | formula               | `{{ID_PREFIX}}-C####`                                                                                                       |
| Type                                                                  | single select         | Content types                                                                                                               |
| Series                                                                | single select         | `{{SERIES}}`                                                                                                                |
| Status                                                                | single select         | Idea, Briefed, Drafting, Assets In Progress, Needs Review, Approved, Scheduled, Published, Archived, Rejected               |
| Priority                                                              | single select         | Priority, Backlog. Empty counts as Backlog. The scheduler fills Priority posts first.                                       |
| Owner                                                                 | single collaborator   |                                                                                                                             |
| Target Platforms                                                      | multi select          | Platform list                                                                                                               |
| Aspect Ratios                                                         | multi select          |                                                                                                                             |
| Brief                                                                 | long text             | What the piece says, to whom, and what the reader should do next. The Copy Writer's main input.                             |
| Copy / Script                                                         | long text             | Master message or script.                                                                                                   |
| Canonical URL                                                         | url                   | The page the posts point to.                                                                                                |
| Review Notes                                                          | long text             | Reviewers and jobs append dated lines.                                                                                      |
| Status Modified By                                                    | last modified by      | Watches Status only. Input to the Content stamp automation.                                                                 |
| Approved By                                                           | single collaborator   | Stamped by automation when a person sets Status to Approved. An API write must never set it.                                |
| Approved At                                                           | date-time             | Same stamp.                                                                                                                 |
| Launch At                                                             | date-time             | When set, every linked post is scheduled at this time. It wins over channel slots.                                          |
| Is Held                                                               | formula               | Below.                                                                                                                      |
| Is CMS Type                                                           | formula               | CMS module. `IF(OR({Type} = "Blog Post", {Type} = "Glossary Term"), 1, 0)`, listing the types in `{{CMS_DOC_TYPES}}`.       |
| CMS Document ID                                                       | single line           | CMS module. The published id, without a draft prefix. Natural key for the inventory sync.                                   |
| CMS Type                                                              | single select         | CMS module. The CMS document type names, for example `blogPost`.                                                            |
| CMS Live                                                              | checkbox              | CMS module. Mirrors the CMS's public flag. A document is public only when it is published **and** live.                     |
| CMS Published At                                                      | date-time             | CMS module.                                                                                                                 |
| CMS Updated At                                                        | date-time             | CMS module.                                                                                                                 |
| Ideas                                                                 | link → Ideas          |                                                                                                                             |
| Platform Posts                                                        | link → Platform Posts |                                                                                                                             |
| Assets                                                                | link → Assets         |                                                                                                                             |
| Episodes, Clips, Articles, Content Angles, Scripts, Newsletter Issues | links                 | One per module that is on.                                                                                                  |
| Posts to Approve                                                      | rollup                | SUM(values) of Platform Posts → Needs Approval.                                                                             |
| Failed Posts                                                          | rollup                | SUM(values) of Platform Posts → Is Failed.                                                                                  |
| Assets to Approve                                                     | rollup                | Core (media). SUM(values) of Platform Posts → Unapproved Assets: draft or unapproved designs and files waiting on a person. |
| Next Manual Step                                                      | formula               | Below.                                                                                                                      |
| Created                                                               | created time          |                                                                                                                             |
| Last Modified                                                         | last modified time    |                                                                                                                             |

The rollups sum a 1/0 formula on the post instead of using a conditional rollup. The API cannot set a rollup's condition, and a plain SUM over a flag is cheaper and easier to check.

**Is Held** marks a series that must not go out until a launch or sign-off (Block D):

```
IF(OR({Series} = "<held series>", {Series} = "<another held series>"), 1, 0)
```

With no held series, the formula is `0`. Releasing a hold is one formula edit, recorded in the brief's change log. Held content may be drafted and approved. The gate stops it.

**Next Manual Step** numbers the human work by urgency. Blank means nothing is waiting on a person.

```
IF(OR({Status} = "Archived", {Status} = "Rejected"), "",
IF(AND({Is Held}, {Status} != "Published"), "9 · Held (" & {Series} & ")",
IF({Failed Posts} > 0, "1 · Fix failed posts",
IF({Status} = "Needs Review",
  IF({Posts to Approve} > 0, "2 · Review content + approve its posts", "2 · Review & approve content"),
IF({Posts to Approve} > 0, "3 · Approve posts",
IF(AND({Status} = "Drafting", OR({Type} = "Promo Video", {Type} = "Script")), "4 · Record voiceover / film",
IF(AND({Status} = "Drafting", {Type} = "Podcast Episode"), "4 · Record / edit episode",
IF({Assets to Approve} > 0, "5 · Finish and approve designs",
IF({Status} = "Assets In Progress", "5 · Finish assets",
IF({Status} = "Drafting", "6 · Finish the draft",
IF({Status} = "Briefed", "6 · Write the draft / script",
IF({Status} = "Idea", "7 · Triage: brief it or reject it",
IF(AND({Status} = "Approved", {Is CMS Type}, NOT({CMS Live})), "8 · Set live in {{CMS | Sanity}}",
"")))))))))))))
```

Remove the podcast line when the long-form module is off, the designs line when the Assets table is off, and the CMS line when the CMS module is off. Each removed line takes one closing parenthesis with it.

## Ideas

Intake. Primary **Idea**. Setting Status to Promoted creates the Content row (automation). Source Ref holds **ids only**, never transcript or message text.

| Field            | Type           | Notes                                                                                                                                                                          |
| ---------------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Idea             | single line    | At most 90 characters. Becomes Content.Title.                                                                                                                                  |
| Idea Number      | autonumber     |                                                                                                                                                                                |
| Idea ID          | formula        | `{{ID_PREFIX}}-I####`                                                                                                                                                          |
| Summary          | long text      | 2–4 sentences: the reader's problem and what the content would explain. Becomes Content.Brief.                                                                                 |
| Status           | single select  | New, Triage, Promoted, Parked, Rejected                                                                                                                                        |
| Source Type      | single select  | Meeting, Recorder Summary, Manual, Analytics, CMS Gap, Request                                                                                                                 |
| Source Ref       | single line    | Ids only: `log:<record id> meeting:<meeting key>`, `summary:<recorder id>`, a CMS id, a post id.                                                                               |
| Redaction Status | single select  | Not Needed, Auto-Redacted, Human-Checked                                                                                                                                       |
| Sensitivity Flag | checkbox       | The source touched people, deals, money, compliance, or something said in confidence. The promote automation refuses it until a person sets Redaction Status to Human-Checked. |
| Topic            | multi select   | `{{TOPICS}}`                                                                                                                                                                   |
| Suggested Type   | single select  | Content types. Copied onto Content.Type.                                                                                                                                       |
| Target Platforms | multi select   | Platform list                                                                                                                                                                  |
| Priority         | single select  | Low, Medium, High                                                                                                                                                              |
| Created By       | single select  | AI, Manual                                                                                                                                                                     |
| Promoted To      | link → Content | Filled by the automation. Its presence makes a second promote a no-op.                                                                                                         |
| Created          | created time   |                                                                                                                                                                                |

## Platform Posts

One row per channel post. Primary **Post Name** (formula).

| Field                                                                                           | Type                  | Notes                                                                                                                                                                                                                       |
| ----------------------------------------------------------------------------------------------- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Post Name                                                                                       | formula               | `{Platform} & " - " & IF({Clip}, {Clip}, IF({Episode}, {Episode}, IF({Content}, {Content}, "Unlinked")))`. Without the long-form module: `{Platform} & " - " & IF({Content}, {Content}, "Unlinked")`.                       |
| Post Number                                                                                     | autonumber            |                                                                                                                                                                                                                             |
| Post ID                                                                                         | formula               | `{{ID_PREFIX}}-P####`                                                                                                                                                                                                       |
| Platform                                                                                        | single select         | Same options as Platform Rules → Platform.                                                                                                                                                                                  |
| Content Mode                                                                                    | single select         | Content modes                                                                                                                                                                                                               |
| Caption                                                                                         | long text             | Never contains hashtags.                                                                                                                                                                                                    |
| Hashtags                                                                                        | long text             | Space-separated, each with `#`. Empty when the channel's Hashtag Style is None. The sender places them.                                                                                                                     |
| First Comment                                                                                   | long text             | Only where the channel supports it.                                                                                                                                                                                         |
| Description                                                                                     | long text             | Long-form video description.                                                                                                                                                                                                |
| Scheduled DateTime                                                                              | date-time             | Blank until the scheduler fills it, unless a person or Launch At set it. No job overwrites a time that is already there.                                                                                                    |
| Approval                                                                                        | single select         | Not Needed, Needs Review, Approved, Changes Requested, Rejected                                                                                                                                                             |
| Copy Approved                                                                                   | checkbox              |                                                                                                                                                                                                                             |
| Creative Approved                                                                               | checkbox              | Tick it for a post with no creative too: it confirms none is needed.                                                                                                                                                        |
| Approval Modified By                                                                            | last modified by      | Watches Approval, Copy Approved, Creative Approved. Input to the stamp automation.                                                                                                                                          |
| Approval Modified At                                                                            | last modified time    | Watches Approval only. The weekly digest flags posts waiting in review longer than `{{REVIEW_LIMIT_DAYS \| 3}}` days.                                                                                                       |
| Approved By                                                                                     | single collaborator   | Stamped only when a person completes full approval. API writes never stamp.                                                                                                                                                 |
| Approved At                                                                                     | date-time             | Same stamp.                                                                                                                                                                                                                 |
| Review Notes                                                                                    | long text             | Required with Changes Requested. Start with `CREATIVE:` when the feedback is for the design only, so the Copy Writer leaves the copy alone.                                                                                 |
| Notes                                                                                           | long text             | Append-only. Jobs write dated lines here.                                                                                                                                                                                   |
| Upload Status                                                                                   | single select         | Pending, Sent, Posted, Error, Missed                                                                                                                                                                                        |
| Delivery Method                                                                                 | single select         | `{{DISTRIBUTOR \| Upload-Post}}`, Manual, RSS, Podcast Host                                                                                                                                                                 |
| Creative Needed                                                                                 | single select         | None, Thumbnail, Quote graphic, Single image, Carousel, Video                                                                                                                                                               |
| Creative Status                                                                                 | single select         | Not needed, Needed, In progress, Done. Needed is the creative step's cue; In progress means a draft design waits for a person.                                                                                              |
| Keywords                                                                                        | long text             |                                                                                                                                                                                                                             |
| Content                                                                                         | link → Content        |                                                                                                                                                                                                                             |
| Assets                                                                                          | link → Assets         |                                                                                                                                                                                                                             |
| Media URLs                                                                                      | lookup                | Assets → Public Media URL. This is what distribution fetches. Repointing one Asset repoints every post that uses it.                                                                                                        |
| Asset Count                                                                                     | count                 | Linked Assets.                                                                                                                                                                                                              |
| Unapproved Assets                                                                               | rollup                | SUM(values) of Assets → Is Unapproved.                                                                                                                                                                                      |
| Creative Ready                                                                                  | formula               | Below. 1 when the post needs no creative, or every linked asset was approved by a person and a Media URL exists.                                                                                                            |
| Has Synthetic Media                                                                             | lookup                | Assets → Synthetic Media. Drives the platform AI label.                                                                                                                                                                     |
| Content ID, Content Title, Content Status, Content Is Held, Content Launch At, Content Priority | lookups               | From Content. The scheduler and distributor read these instead of a second query.                                                                                                                                           |
| Needs Approval                                                                                  | formula               | `IF({Approval} = "Needs Review", 1, 0)`                                                                                                                                                                                     |
| Is Failed                                                                                       | formula               | `IF(OR({Upload Status} = "Error", {Upload Status} = "Missed"), 1, 0)`                                                                                                                                                       |
| Episode, Clip                                                                                   | links                 | Long-form module.                                                                                                                                                                                                           |
| Clip Content Score                                                                              | lookup                | Long-form module. Clip → Content Score.                                                                                                                                                                                     |
| Asset Source                                                                                    | single select         | Long-form module. Full Episode, Clip, Article                                                                                                                                                                               |
| Content Angle                                                                                   | link → Content Angles | Angles module.                                                                                                                                                                                                              |
| Thumbnail                                                                                       | attachments           |                                                                                                                                                                                                                             |
| Platform Post URL                                                                               | url                   | The live URL from the distributor's response, or pasted by the person who posted by hand. Never typed from a guess.                                                                                                         |
| Sent At                                                                                         | date-time             | Written by Distribute just before the request. Confirm Posts measures a stuck post from it.                                                                                                                                 |
| Distributor Request ID                                                                          | single line           |                                                                                                                                                                                                                             |
| Distributor Job Status                                                                          | single select         | Not Sent, Queued, In Progress, Completed, Partial, Failed. Partial: some targets succeeded, or a success came back without the platform's own result. A person checks the channel.                                          |
| Error Details                                                                                   | long text             | Scrubbed. No tokens, no headers.                                                                                                                                                                                            |
| Retry Count                                                                                     | number                |                                                                                                                                                                                                                             |
| Published At                                                                                    | date-time             |                                                                                                                                                                                                                             |
| Views, Likes, Comments, Shares, Saves, Clicks                                                   | number                | Written by the analytics pull.                                                                                                                                                                                              |
| Avg View %                                                                                      | number                | 0–100.                                                                                                                                                                                                                      |
| Engagement Rate                                                                                 | percent               | Written by the analytics pull.                                                                                                                                                                                              |
| Engagement Score (1–10)                                                                         | single select         | 1 — Very low, 2 — Low, 3 — Below average, 4 — Slightly below average, 5 — Average, 6 — Slightly above average, 7 — Good, 8 — Strong, 9 — Excellent, 10 — Exceptional. Written by the analytics pull. People do not edit it. |
| Metrics Updated At                                                                              | date-time             |                                                                                                                                                                                                                             |
| Created Time                                                                                    | created time          |                                                                                                                                                                                                                             |
| Last Modified Time                                                                              | last modified time    |                                                                                                                                                                                                                             |

**Creative Ready**:

```
IF(OR({Creative Needed} = "None", LEN({Creative Needed} & '') = 0), 1,
  IF(AND({Asset Count} > 0, {Unapproved Assets} = 0, LEN({Media URLs} & '') > 0), 1, 0))
```

### The gate

Nothing is sent to the distributor unless **every** check holds at send time:

1. Approval = Approved.
2. Copy Approved and Creative Approved are both ticked.
3. Approved By is set, and its email is in the channel's Allowed Approvers. An unstamped approval never passes, so the final approval is always a human click.
4. On a personal account, Approved By is the Account Owner. Nobody approves for someone else's own profile.
5. Content Status is Approved or Scheduled.
6. Content Is Held is 0.
7. The caption fits the channel's Max Caption Length once hashtags are placed (see Hashtag Style).
8. The channel's Platform Rules row is active, and the Content Mode is in its Supported Content Modes.
9. A mode that carries media has at least one Media URL. A video is no longer than Max Video Seconds, when that is set.
10. Creative Ready is 1. When the post needs creative, every linked asset was approved by a person: its own stamp, or, for a page of a multi-page design, its parent design's stamp. A draft design, a superseded file, or an export link never goes out.
11. The distributor's profile listing, read this run, holds the channel's Distributor Profile with the platform connected. A missing profile or an unconnected platform is never guessed around.
12. When Has Synthetic Media is true and the channel's AI Label Required is ticked, the distributor can set that platform's AI label. If it cannot, the gate refuses the post, and a person posts it by hand with the label (Delivery Method = Manual).
13. Long-form module: a post linked to a clip needs Clip Content Score of at least `{{CLIP_SCORE_MIN | 5}}`.
14. A `[TEST]` title goes only to `{{DISTRIBUTOR_TEST_PROFILE | none}}`. With no test profile it is never sent.

Rows imported from an older system that approved posts by script carry no stamp, so they never pass. That is intended: a person re-approves the ones that should still go out.

A post that fails a check stays Pending and gets one dated Notes line naming the check, written once. Distribute checks it again on every run: it goes out as soon as every check holds (a person re-approves, an account is reconnected), or goes Missed when its window passes. Distribution re-checks the whole gate at send time, because approvals can be withdrawn after scheduling.

### Parking

To park a post: Approval = Not Needed, and Review Notes starts with `PARKED <YYYY-MM-DD> (<who>): <reason>`. To revive it, set Approval back to Needs Review. A repeat of a post that is already live gets the same treatment with `STALE` instead of `PARKED`. The label starts at the first character, because views and formulas match on it.

## Platform Rules

One row per channel. **Adding a channel is a row, not a code change.** Primary **Platform Name**.

| Field                   | Type               | Notes                                                                                                                                                                                                             |
| ----------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Platform Name           | single line        | Human label, for example "Company LinkedIn page".                                                                                                                                                                 |
| Platform                | single select      | Platform list                                                                                                                                                                                                     |
| Is Active               | checkbox           | Off: the channel is out of the plan. No drafts, nothing sent.                                                                                                                                                     |
| Delivery Method         | single select      | `{{DISTRIBUTOR \| Upload-Post}}`, Manual. Manual: drafted and approved here, posted by a person (the platform forbids API posting, or the account is not connected yet). Copied onto each new post.               |
| Distributor Key         | single select      | The distributor's lowercase platform value. Upload-Post: instagram, facebook, linkedin, x, tiktok, youtube, threads, bluesky, pinterest, google_business.                                                         |
| Distributor Profile     | single select      | The distributor profile that holds this account, spelled exactly as the distributor's profile listing names it (Upload-Post: `list_users`). One for the brand, one per personal account. Never typed from memory. |
| Platform ID             | single line        | The id the distributor needs (page, organisation, location). Store LinkedIn organisations as `urn:li:organization:<id>` and strip the prefix where an API wants the bare id.                                      |
| Platform Account ID     | single line        | A parent id, when the platform needs one.                                                                                                                                                                         |
| Account Owner           | single select      | `Brand`, or the person who owns a personal account.                                                                                                                                                               |
| Account Owner Email     | email              | Personal accounts only. The gate compares Approved By with it.                                                                                                                                                    |
| Allowed Approvers       | multi select       | Approver emails, lowercase. Emails, because names are not unique and the collaborator field returns the email reliably.                                                                                           |
| Frequency               | single select      | Daily, 3x Weekly, 2x Weekly, Weekly                                                                                                                                                                               |
| Posting Days            | multi select       | Weekdays. Empty: the scheduler derives them from Frequency.                                                                                                                                                       |
| Posting Time (Local)    | single line        | 24-hour `HH:mm` in `{{TIME_ZONE}}`.                                                                                                                                                                               |
| Max Caption Length      | number             | Caption plus placed hashtags. Defaults per platform: the table in `workflows/distribute.md`. YouTube Shorts title 100, X 280.                                                                                     |
| Max Video Seconds       | number             | Empty: no pipeline cap.                                                                                                                                                                                           |
| Default Aspect Ratio    | single select      |                                                                                                                                                                                                                   |
| Supported Content Modes | multi select       | Content modes                                                                                                                                                                                                     |
| Hashtag Style           | single select      | `None`, `Appended (3-5)`, `First comment (1-3)`. Below.                                                                                                                                                           |
| First Comment Supported | checkbox           |                                                                                                                                                                                                                   |
| AI Label Required       | checkbox           | The platform requires a label on realistic synthetic media.                                                                                                                                                       |
| Caption Style Notes     | long text          | Read first by the Copy Writer. Overrides its channel defaults. Record platform quirks here, for example "links are stripped on this account: no URL in the caption or first comment".                             |
| Preferred Content Mix   | long text          |                                                                                                                                                                                                                   |
| Post All Episodes       | checkbox           | Long-form module. Episode content targets this channel by default.                                                                                                                                                |
| Post All Clips          | checkbox           | Long-form module. Clip content targets this channel by default.                                                                                                                                                   |
| CTA Type                | single select      | Learn More, Book, Sign Up, Order, Shop, Call. For channels with a button (Google Business).                                                                                                                       |
| CTA URL                 | url                |                                                                                                                                                                                                                   |
| Notes                   | long text          |                                                                                                                                                                                                                   |
| Last Modified           | last modified time |                                                                                                                                                                                                                   |

**Hashtag Style.** The writer always puts hashtags in the Hashtags field. The sender places them:

| Style                 | Writer                | Sender                                                         | Fit check                    |
| --------------------- | --------------------- | -------------------------------------------------------------- | ---------------------------- |
| `None`                | Leaves Hashtags empty | Sends the caption only                                         | Caption ≤ Max                |
| `Appended (3-5)`      | Writes 3–5            | Caption, a blank line, then the hashtags                       | Caption + 2 + Hashtags ≤ Max |
| `First comment (1-3)` | Writes 1–3            | First Comment, a blank line, then the hashtags, as the comment | Caption ≤ Max                |

In the system this kit came from, one "0" option meant "in the caption" on one channel and "no hashtags" on another, and the writer and the sender read it differently. Keep the three styles distinct.

## Assets

One produced file or design. Primary **Asset Name**: `<Content Title> · <Kind> · v<Version>`, with the platform after the title when the asset serves one channel (`<Content Title> · <Platform> · <Kind> · v<Version>`). A person approves every asset that reaches a post; the gate reads the stamp, not the Status alone.

| Field              | Type                  | Notes                                                                                                                                                                                         |
| ------------------ | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Asset Name         | single line           |                                                                                                                                                                                               |
| Asset Number       | autonumber            |                                                                                                                                                                                               |
| Content            | link → Content        |                                                                                                                                                                                               |
| Platform Posts     | link → Platform Posts |                                                                                                                                                                                               |
| Kind               | single select         | Video, Image, Carousel, Carousel Slide, Carousel PDF, Voiceover Audio, Music, Thumbnail, Script, Caption File. Carousel is a multi-page design; its exported pages are Carousel Slide assets. |
| Tool               | single select         | The tools from Block G (default `{{DESIGN_TOOL \| Canva}}`, Descript, ElevenLabs), plus Manual                                                                                                |
| External ID        | single line           | Design id, editor project id, or voice history id.                                                                                                                                            |
| Template           | link → Templates      | Creative module. The brand template a draft was made from.                                                                                                                                    |
| Export Format      | single select         | PNG, JPG, PDF, MP4                                                                                                                                                                            |
| Page               | number                | Empty for a whole file or design. The page number for a page of a multi-page design.                                                                                                          |
| Parent Asset       | link → Assets         | A page's design asset. Create it from this side; Airtable adds the inverse (`Pages`).                                                                                                         |
| Parent Approved By | lookup                | Parent Asset → Approved By. A page is approved through its parent's human stamp.                                                                                                              |
| Edit URL           | url                   |                                                                                                                                                                                               |
| File Path          | single line           | Path under `{{MEDIA_ROOT}}` in `{{FILE_HOST \| Dropbox}}`.                                                                                                                                    |
| Public Media URL   | url                   | A direct download the distributor can fetch. Never an Airtable attachment URL: those expire within hours.                                                                                     |
| Link Checked At    | date-time             | When a ranged GET last proved the link returns media.                                                                                                                                         |
| Preview            | attachments           |                                                                                                                                                                                               |
| Aspect Ratio       | single select         |                                                                                                                                                                                               |
| Width, Height      | number                |                                                                                                                                                                                               |
| Duration (s)       | number                |                                                                                                                                                                                               |
| Version            | number                | A new export is a new version with a new file next to the old one. Notes keep the old URL for rollback.                                                                                       |
| Synthetic Media    | checkbox              | Cloned voice, generated imagery, or generated video of a realistic person or scene.                                                                                                           |
| Status             | single select         | Requested, Draft, In Progress, Ready for Review, Approved, Superseded. Draft: made by a job, untouched by a person. Approved only by a person's action.                                       |
| Status Modified By | last modified by      | Watches Status only. Input to the asset stamp automation.                                                                                                                                     |
| Approved By        | single collaborator   | Stamped by automation when a person sets Status to Approved. An API write never stamps.                                                                                                       |
| Approved At        | date-time             | Same stamp. A design changed after this time is approved again before it is exported.                                                                                                         |
| Is Unapproved      | formula               | Below. 1 blocks every post the asset is linked to.                                                                                                                                            |
| Notes              | long text             |                                                                                                                                                                                               |

**Is Unapproved**:

```
IF(AND({Status} = "Approved", OR(LEN({Approved By} & '') > 0, LEN({Parent Approved By} & '') > 0)), 0, 1)
```

A superseded or draft asset left linked to a post blocks it: unlink it. The creative routine does that itself when it makes a new version.

## Pipeline Errors

One failed run or item. Primary **Summary**. Never store secrets, request headers, transcript text, or message bodies.

| Field         | Type          | Notes                                                               |
| ------------- | ------------- | ------------------------------------------------------------------- |
| Summary       | single line   | One line a person can act on.                                       |
| Source        | single select | Workflow, Agent Routine, Airtable Automation, Manual                |
| Workflow      | single line   | Job name.                                                           |
| Workflow ID   | single line   |                                                                     |
| Execution ID  | single line   |                                                                     |
| Execution URL | url           |                                                                     |
| Failed Node   | single line   |                                                                     |
| Error Message | long text     | Scrubbed, at most 2,000 characters.                                 |
| Record Refs   | single line   | Human ids and record ids, for example `{{ID_PREFIX}}-P0042 rec…`.   |
| Occurrences   | number        | The error handler counts repeats here instead of creating new rows. |
| Occurred At   | date-time     |                                                                     |
| Status        | single select | New, Investigating, Resolved, Ignored                               |
| Notes         | long text     |                                                                     |

A new row emails `{{ALERT_EMAIL | the owner}}`.

## Scores

**Clip Content Score** (long-form module), 1–10, typed by a person before a clip posts. A routine may suggest a score in Score Notes. It never writes the score.

| Criterion    | 0                               | 1                       | 2                                 |
| ------------ | ------------------------------- | ----------------------- | --------------------------------- |
| Hook         | Nothing in the first 3 s        | A slow start            | Stops the scroll in the first 3 s |
| Standalone   | Needs the episode to make sense | Mostly clear            | Complete on its own               |
| Relevance    | Off the audiences in the brief  | Adjacent                | Squarely for an audience          |
| Brand safety | A hard copy rule is at risk     | Needs a careful caption | Clean                             |
| Production   | Audio or framing problems       | Acceptable              | Clean audio, captions, framing    |

Score = the sum, with a floor of 1. A brand-safety score of 0 caps the total at 4. Under `{{CLIP_SCORE_MIN | 5}}` goes back to review and the gate will not send it.

**Engagement Score** (Platform Posts), 1–10, written by the analytics pull after posting. It needs `{{SCORE_MIN_VIEWS | 50}}`+ views and `{{SCORE_MIN_HOURS | 72}}` hours since Published At. It compares a post with **its own channel's** history, so 5 means typical for us on that channel, not typical for the platform. **Back to review:** when every scored post of one Content row (at least two) is `{{BACK_TO_REVIEW_MAX_SCORE | 3}}` or lower, its remaining approved, unsent posts go back to Needs Review. The rule lives in the analytics job, not in a field people edit. The calculation is in [`workflows/analytics-pull.md`](workflows/analytics-pull.md).

## Module: Source Recordings (meetings)

The opt-in log the Meeting Idea Miner reads. It may live in this base or in an existing operations base (Block E). One row per recording file. Primary **Recording Key**.

| Field            | Type          | Notes                                                                                                                                                                                     |
| ---------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Recording Key    | single line   | Natural key `{{RECORDER_KEY_PREFIX \| rec}}:<transcript id>`, or `file:<file host id>` for a file with no recorder id.                                                                    |
| Meeting Key      | single line   | Groups the files of one meeting: `cal:<iCalUID>#<start date>` (recurring occurrences share an iCalUID) or `{{RECORDER_KEY_PREFIX \| rec}}:<transcript id>`. `TEST-` marks a test meeting. |
| Recording Type   | single select | Transcript, Captions, Summary, Audio, Video                                                                                                                                               |
| Recording Date   | date-time     | Meeting start.                                                                                                                                                                            |
| File Path        | single line   | Path on the file host, usually under `{{TRANSCRIPT_ROOT \| /Meeting Archive}}`.                                                                                                           |
| File ID          | single line   | File host id.                                                                                                                                                                             |
| Meeting Topic    | single line   | **Never read by a routine.** Topics carry names.                                                                                                                                          |
| Use for Content  | checkbox      | The opt-in. Ticked by a person, never by a job.                                                                                                                                           |
| Content Mined At | date-time     | Full ISO date-time, stamped by the miner.                                                                                                                                                 |
| Ideas Created    | number        | Stamped by the miner.                                                                                                                                                                     |
| Notes            | long text     |                                                                                                                                                                                           |

Rows come from a recording-archive workflow, the recorder's integration, or a person. The interview says which. If the Sales Pipeline kit already archives transcripts to file storage, list that archive folder: never read its CRM tables.

## Module: Episodes and Clips (long-form series)

**Episodes.** Primary **Episode ID** (formula).

| Field                                                                                | Type               | Notes                                                                                  |
| ------------------------------------------------------------------------------------ | ------------------ | -------------------------------------------------------------------------------------- |
| Episode ID                                                                           | formula            | `{{ID_PREFIX}}-E###`                                                                   |
| Title, Subtitle                                                                      | single line        |                                                                                        |
| Episode Number                                                                       | number             |                                                                                        |
| Status                                                                               | single select      | Draft, Recorded, Edited, Ready for Distribution, Published                             |
| Speakers                                                                             | multi select       | Hosts from the brief, plus Guest                                                       |
| Guest Name                                                                           | single line        | Only with the guest's consent to be named.                                             |
| Themes                                                                               | multi select       | `{{TOPICS}}`                                                                           |
| Recording Date, Publish Date                                                         | date               |                                                                                        |
| Master URL                                                                           | url                | The file host's share link. The link fixer rewrites it to a direct download.           |
| Recording Link                                                                       | url                | The recording tool's project.                                                          |
| Full Transcript                                                                      | long text          |                                                                                        |
| Chapters                                                                             | long text          |                                                                                        |
| Short Summary (50 words)                                                             | long text          |                                                                                        |
| Medium Summary (150 words)                                                           | long text          |                                                                                        |
| Key Quotes                                                                           | long text          | Published speech only. The Copy Writer may quote these.                                |
| SEO Keywords                                                                         | long text          |                                                                                        |
| Show Notes                                                                           | long text          |                                                                                        |
| Long-form Video URL, Podcast Host URL                                                | url                |                                                                                        |
| Podcast Host Status                                                                  | single select      | Draft, Ready to Upload, Uploaded, Published                                            |
| Thumbnail                                                                            | attachments        |                                                                                        |
| Thumbnail Template URL                                                               | url                |                                                                                        |
| Is Recent                                                                            | formula            | `IF(IS_AFTER({Publish Date}, DATEADD(TODAY(), -{{RECENT_DAYS \| 30}}, 'days')), 1, 0)` |
| Generate Posts                                                                       | checkbox           | Ticked by a person. Creates the episode's Content row.                                 |
| Clips, Content, Platform Posts, Articles, Content Angles, Scripts, Newsletter Issues | links              | Per module.                                                                            |
| Last Modified                                                                        | last modified time |                                                                                        |

**Clips.** Primary **Clip ID** (formula).

| Field                    | Type                  | Notes                                                                  |
| ------------------------ | --------------------- | ---------------------------------------------------------------------- |
| Clip ID                  | formula               | `<Episode ID>-C##`                                                     |
| Clip Title               | single line           |                                                                        |
| Episode                  | link → Episodes       |                                                                        |
| Clip Number              | number                |                                                                        |
| Status                   | single select         | Draft, Needs Score, Ready to Schedule, Scheduled, Posted, Not Approved |
| File URL                 | url                   | Share link. The link fixer rewrites it to a direct download.           |
| Hook                     | single line           |                                                                        |
| Clip Transcript          | long text             |                                                                        |
| Clip Duration (seconds)  | number                |                                                                        |
| Clip Category            | single select         | Problem, Solution, Lesson, Story, Hot take, Insight, Call to action    |
| CTA                      | single line           |                                                                        |
| Key Quote                | single line           |                                                                        |
| Key Takeaway             | long text             | Becomes the Brief of the clip's Content row.                           |
| Content Score            | number                | 1–10, see Scores.                                                      |
| Score Notes              | long text             |                                                                        |
| Scored At                | date                  |                                                                        |
| Best Performing Platform | single select         | Platform list                                                          |
| Times Resurfaced         | number                |                                                                        |
| Last Resurfaced Date     | date                  |                                                                        |
| Generate Posts           | checkbox              | Ticked by a person. Creates the clip's Content row.                    |
| Content                  | link → Content        |                                                                        |
| Content Angle            | link → Content Angles |                                                                        |
| Platform Posts           | link → Platform Posts |                                                                        |
| Notes                    | long text             |                                                                        |

## Module: Content Angles

Hook options for a piece, written by the Copy Writer. Primary **Angle Name** (typed). **Angle Label** (formula):

```
IF({Clip}, {Clip} & " — " & {Angle Type},
  IF({Episode}, {Episode} & " — " & {Angle Type},
    IF({Content}, {Content} & " — " & {Angle Type},
      "UNLINKED — " & {Angle Type})))
```

Remove the Clip and Episode lines (and their parentheses) without the long-form module.

| Field                                  | Type          | Options                                                                                                                                                            |
| -------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Angle Type                             | single select | Thought leadership, Contrarian take, Practical tip, Story / personal lesson, Myth vs reality, Checklist / framework, Quote-based, Thread outline, Carousel outline |
| Angle Status                           | single select | Needs AI, Needs Review, Approved, Converted to Posts                                                                                                               |
| Goal                                   | single select | Education, Authority, Growth, Trust, Brand, Conversion                                                                                                             |
| Priority                               | single select | Low, Medium, High                                                                                                                                                  |
| Generated By                           | single select | AI, Manual                                                                                                                                                         |
| Hook A, Hook B, Hook C                 | single line   | At most 120 characters each.                                                                                                                                       |
| Core Post (Text-only)                  | long text     | At most 600 characters, no hashtags.                                                                                                                               |
| Carousel Outline                       | long text     |                                                                                                                                                                    |
| Thread Outline                         | long text     |                                                                                                                                                                    |
| Content, Clip, Episode, Platform Posts | links         |                                                                                                                                                                    |

## Module: Scripts

Video and voiceover scripts. Primary **Script Title**.

| Field                        | Type          | Notes                                                                                             |
| ---------------------------- | ------------- | ------------------------------------------------------------------------------------------------- |
| Format                       | single select | From Block G: `Solo — <Person>` per presenter, `Dialogue — <Person> + <Person>`                   |
| Video Category               | single select | Educational, Entertainment, Founder Story, Hot Take, Community Spotlight, Trend Reaction          |
| Status                       | single select | Draft, Needs Review, Approved, Filmed, Posted                                                     |
| Voice                        | single select | `<Person> (cloned)` per consenting voice owner, and None. None means recorded live, not a clone.  |
| Target Platforms             | multi select  | Platform list                                                                                     |
| Hook                         | single line   |                                                                                                   |
| Script Body                  | long text     |                                                                                                   |
| Estimated Duration (seconds) | number        |                                                                                                   |
| AI Generated                 | checkbox      |                                                                                                   |
| Timed Lines                  | long text     | JSON array `[{"start": s, "end": s, "text": "..."}]`. Feeds a teleprompter or voiceover assembly. |
| Notes                        | long text     |                                                                                                   |
| Episode, Content             | links         |                                                                                                   |

A cloned voice needs the voice owner's written consent, recorded in the brief, before its option exists.

## Module: Articles

Long-form drafted outside the CMS. Primary **Title**.

| Field                      | Type          | Notes                                                                                                        |
| -------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------ |
| Status                     | single select | Draft, Needs AI, Needs Review, Ready to Publish, Published                                                   |
| Slug                       | single line   | Kebab-case, at most 64 characters.                                                                           |
| SEO Keyword                | single line   |                                                                                                              |
| Meta Title                 | single line   |                                                                                                              |
| Meta Description           | long text     |                                                                                                              |
| Outline                    | long text     |                                                                                                              |
| Body (Markdown)            | long text     |                                                                                                              |
| Body (CMS JSON)            | long text     | The CMS's block format, for example Portable Text for Sanity. Wins over Markdown when it parses.             |
| Feature Image              | attachments   | Copied to the CMS in the same run: attachment URLs expire.                                                   |
| Feature Image Template URL | url           |                                                                                                              |
| Publish Date               | date          |                                                                                                              |
| Pull Quote                 | long text     |                                                                                                              |
| Notes                      | long text     | Labelled lines at the first character are read by the article routine: `Author: <name>`, `Category: <slug>`. |
| CMS Document ID            | single line   |                                                                                                              |
| CMS URL                    | url           | The CMS editor link for the draft.                                                                           |
| Episode, Content           | links         |                                                                                                              |

## Module: Newsletter Issues

Primary **Issue Title**. A person schedules each issue in the email service (`{{ESP | Resend}}`). No job in this kit sends email to subscribers.

| Field                            | Type          | Notes                                              |
| -------------------------------- | ------------- | -------------------------------------------------- |
| Issue Number                     | number        |                                                    |
| Format                           | single select | Deep Dive, Roundup, Story, Hot Take                |
| Status                           | single select | Draft, Needs Review, Approved, Scheduled, Sent     |
| Scheduled Send Date              | date-time     |                                                    |
| Subject Line, Preview Text       | single line   |                                                    |
| Subject Line Options             | long text     |                                                    |
| Opening Hook, Body, Key Takeaway | long text     |                                                    |
| CTA                              | single line   |                                                    |
| CTA URL                          | url           |                                                    |
| Word Count                       | number        |                                                    |
| Broadcast URL                    | url           | The issue in the email service.                    |
| Open Rate, Click Rate            | percent       | Typed from the email service, or synced read-only. |
| Notes                            | long text     |                                                    |
| Source Episodes, Content         | links         |                                                    |

## Module: Creative (Templates)

One row per brand template the creative step may use: one per channel and kind. Primary **Name** (`<Platform> · <Type> · <Dimension>`). The `CP · Creative Drafts` routine reads it on every run. A person adds and maintains rows; the routine never writes here.

| Field             | Type          | Notes                                                                                                                                                                                                                                       |
| ----------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name              | single line   |                                                                                                                                                                                                                                             |
| Type              | single select | Single image, Quote graphic, Carousel, Thumbnail (spelled as on Platform Posts → Creative Needed), plus Story, Cover, End card for people's own use                                                                                         |
| Dimension         | single select | 1080×1350, 1080×1080, 1080×1920, 1920×1080, 1280×720, 1200×1200, 1600×900, 1200×900                                                                                                                                                         |
| Destination       | multi select  | Platform list. The channels this template serves.                                                                                                                                                                                           |
| Design Tool       | single select | From Block G                                                                                                                                                                                                                                |
| Brand Template ID | single line   | The design tool's brand template id. Drafts are new designs made from it; the template itself is never edited by a job.                                                                                                                     |
| Has Data Fields   | checkbox      | Ticked when the design tool lists autofill fields for the template (read at build, and when the template changes).                                                                                                                          |
| Field Map         | long text     | One line per field: `<field> = <source>[, <fallback>] [max <n>]`. The field is an autofill field name, or the exact placeholder text in the template. Sources: `title`, `hook`, `caption_line`, `quote`, `slides`, `url`, `literal:<text>`. |
| Export Format     | single select | PNG, JPG, PDF, MP4                                                                                                                                                                                                                          |
| Is Active         | checkbox      | Off: the routine does not use it.                                                                                                                                                                                                           |
| Design ID         | single line   | The master design behind the brand template.                                                                                                                                                                                                |
| Edit URL          | url           |                                                                                                                                                                                                                                             |
| Template Assets   | attachments   | A preview.                                                                                                                                                                                                                                  |
| Assets            | link → Assets | The inverse of Assets → Template.                                                                                                                                                                                                           |
| Notes             | long text     |                                                                                                                                                                                                                                             |

Example Field Map for a quote graphic:

```
QUOTE TEXT = quote [max 140]
ATTRIBUTION = literal:From the podcast
HANDLE = url
```

## Views

A person creates views in the UI (`MANUAL.md`). Views marked **do not filter, do not delete** are watched by an automation: a filter on them silently stops the automation.

| Table             | Section  | Views                                                                                                                                                                                                                          |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Content           | My Queue | Action Needed (Next Manual Step not empty, sorted by it), Review (Needs Review), Posts to Approve (> 0), Record/Produce (Next Manual Step starts with 4 or 5), Failed Posts (> 0)                                              |
| Content           | Planning | Pipeline Board (kanban by Status), Launch Calendar (calendar on Launch At), Held (Is Held = 1)                                                                                                                                 |
| Content           | Library  | Published, Archived & Rejected, All Content (unfiltered)                                                                                                                                                                       |
| Platform Posts    |          | Needs Approval, Notes & Holds (Review Notes starts with PARKED or STALE), Pending Schedule (approved, no time), Scheduled Today, Scheduled This Week, Calendar, Sent, Live in Last 7 Days, Errors & Holds, Master (unfiltered) |
| Ideas             |          | Triage (New or Triage, by Priority), Sensitive (Sensitivity Flag on, not Human-Checked), Promoted, All                                                                                                                         |
| Pipeline Errors   |          | Open (New or Investigating), All                                                                                                                                                                                               |
| Assets            |          | Designs to Finish (Draft or In Progress), Ready for Review, Approved Without Media (Approved, Page empty, Public Media URL empty, no Pages), Superseded, All                                                                   |
| Templates         |          | Active (by Destination), All                                                                                                                                                                                                   |
| Source Recordings |          | Opted In, Not Mined                                                                                                                                                                                                            |
| Episodes          |          | **Master — do not filter, do not delete** (link fixer and publish automation watch it), In Production, Needs Thumbnail                                                                                                         |
| Clips             |          | **Master — do not filter, do not delete** (link fixer watches it), Needs Score, Back Catalog — Ready to Repurpose (Posted, Content Score ≥ 7, not resurfaced in 90 days)                                                       |

Master views exist unfiltered on purpose: they are what automations and people use to find a row that every other view hides.

## Automations

Create each as a draft, off. A person turns it on after a test record proves it (`MANUAL.md`).

| Name                                               | Trigger                                                                                                    | Does                                                                                                                                                                                                                                                                            |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ideas: Promote to Content                          | Ideas.Status = Promoted, Promoted To empty, and (Sensitivity Flag off or Redaction Status = Human-Checked) | Create Content: Status Briefed, Title = Idea, Brief = Summary, Type = Suggested Type, Target Platforms = Target Platforms. Link it in Promoted To. A second save does nothing.                                                                                                  |
| Review: stamp Platform Post approval               | Approval, Copy Approved, or Creative Approved changes                                                      | Full approval by a person: stamp Approved By and Approved At. Changes Requested or Rejected: clear both stamps and untick both boxes, so the revision is reviewed again. Anything else: clear the stamps. Never sets Approval.                                                  |
| Review: stamp Content approval                     | Content.Status changes                                                                                     | Approved by a person: stamp. A move back to Idea, Briefed, Drafting, Assets In Progress, Needs Review, or Rejected clears it. Scheduled, Published, and Archived keep it.                                                                                                       |
| Review: stamp Asset approval                       | Assets.Status changes                                                                                      | Approved by a person: stamp Approved By and Approved At. Any other status clears both. Never sets Status. Built the same way as the post stamp (below).                                                                                                                         |
| Review: email when Content needs review            | Status becomes Needs Review                                                                                | Email `{{REVIEW_EMAIL_TO}}`: Content ID, Title, Next Manual Step, and the Content Review interface link.                                                                                                                                                                        |
| Pipeline Errors: email on new row                  | Record created                                                                                             | Email `{{ALERT_EMAIL \| the owner}}`: Summary, Workflow, Execution URL, Record Refs. Nothing else.                                                                                                                                                                              |
| Weekly digest                                      | `{{DIGEST_DAY_TIME \| Monday 09:00}}`                                                                      | Find posts published in the last 7 days with their scores, open errors, the review queue (flagging posts whose Approval Modified At is older than `{{REVIEW_LIMIT_DAYS \| 3}}` days), the next 7 days of scheduled posts, and held content. Email `{{DIGEST_TO \| the owner}}`. |
| Long-form: create Content from a clip              | Clips.Generate Posts ticked and Content empty                                                              | Create Content: Type Clip, Status Briefed, Title = Clip Title, Brief = Key Takeaway, links to the clip and its episode.                                                                                                                                                         |
| Long-form: create Content from an episode          | Episodes.Generate Posts ticked and Content empty                                                           | Same, Type Podcast Episode, Brief = Medium Summary.                                                                                                                                                                                                                             |
| Long-form: mark episode Published                  | On Episodes → Master: Long-form Video URL and Podcast Host URL both filled                                 | Status = Published, Podcast Host Status = Published.                                                                                                                                                                                                                            |
| Media link fixer (only if the file host needs one) | On Episodes → Master or Clips → Master: the share URL changes                                              | Rewrite the share link to its direct-download form, on **the same record** that triggered it. Dropbox: `dl=0` becomes `dl=1`.                                                                                                                                                   |

The weekly digest runs on the automation's own clock. If that clock is UTC, the local send time moves by an hour at each daylight-saving change. Pick a time where that does not matter and say so in the brief.

A link-fixer script once wrote an episode's id into the Clips table and failed on every run. Each fixer updates only the record in its own trigger.

### Approval stamps: how to build them

The stamps are the approval gate. A stamp must be impossible to produce from the API.

- Agents and workflows usually write with the same Airtable account a person uses, so "who changed this" can name a person even when an agent did it. Build the stamp on something only a human can do. Two designs work:
  1. **Interface button (preferred where available).** Content Review gets an Approve button whose automation trigger is the button click. The automation stamps the person who clicked and sets nothing else.
  2. **Field watch with a dedicated integration account.** Agents and workflows write through a separate Airtable user (or a token owned by one) that is never in any Allowed Approvers list. The automation stamps Approved By from Approval Modified By only when that collaborator is not the integration account.
- Both checks downstream hold anyway: the gate requires Approved By to be in the channel's Allowed Approvers, and an integration account never is.
- Assets use the same design: an **Approve design** button on the Asset Review page, or the field watch on Status. A job may set an asset's Status, but only a person's action stamps it, and the gate reads the stamp.
- Test it (Phase 7): approve a `[TEST]` post in the UI and see your own stamp. Then set Approval to Approved through the API and confirm Approved By stays empty. Do the same for a `[TEST]` asset. If an API write stamps, stop. Do not weaken the gate to make the test pass.

## Interfaces

Drafted by the agent where the tool allows, finished and shared by a person.

**Content Review** (shared with every approver):

| Page         | Type | Source                                                                                                                                                          |
| ------------ | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Review Queue | list | Content in Needs Review, detail page with its linked posts                                                                                                      |
| Post Queue   | list | Platform Posts in Needs Review: Caption, Hashtags, Media URLs, both checkboxes, Approval, Review Notes                                                          |
| Asset Review | list | Assets in Draft, In Progress, or Ready for Review: Preview, Edit URL, Status, the linked post's Caption, Notes. Detail page with the **Approve design** button. |
| Calendar     | list | Platform Posts by Scheduled DateTime, post detail page                                                                                                          |
| Inventory    | list | All Content, same detail page                                                                                                                                   |

**Content Command Center** (dashboard): big numbers for Action Needed, Posts to Approve, Designs to Finish, Pending Schedule, and Open Errors. Lists: Upcoming Posts (next 7 days), Failed Posts, and Top Posts by Engagement Score (last 30 days). Long-form module: Episodes in Production and Clips Needing a Score.
