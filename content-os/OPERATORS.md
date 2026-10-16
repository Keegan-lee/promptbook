# {{COMPANY}} Content OS — operator's manual

> Build agent: fill this from the brief at the end of the build, remove the sections for modules that are off, delete this note, and hand it to the people named in Block K. Every link must be live.

Base: [{{BASE_NAME}}]({{URL_BASE}}). Brief: {{BRIEF_LOCATION}}. Brand pack: {{BRAND_PACK_LOCATION}} (v{{BRAND_PACK_VERSION}}).

Search rows by **{{ID_PREFIX}}-C####** (content), **{{ID_PREFIX}}-P####** (posts), and **{{ID_PREFIX}}-I####** (ideas). Nobody can search by Airtable record id, so never ask for one.

## Where to click

| You want to                                                | Open                                                                |
| ---------------------------------------------------------- | ------------------------------------------------------------------- |
| See what is waiting on a person                            | Content → **Action Needed**, or the Next Manual Step field          |
| Approve a piece and its posts                              | Interface [Content Review]({{URL_CONTENT_REVIEW}})                  |
| Finish or approve a design                                 | Content Review → Asset Review                                       |
| See why an approved post has not gone out                  | The post's Notes: the newest `Gate:` line names the check           |
| See the week                                               | Content Review → Calendar, or Content → Launch Calendar             |
| See the whole pipeline at a glance                         | Interface [Content Command Center]({{URL_COMMAND_CENTER}})          |
| Triage ideas                                               | Ideas → Triage                                                      |
| See a failed post                                          | Content → Failed Posts, or Platform Posts → Errors & Holds          |
| Change cadence, posting time, caption length, or approvers | Platform Rules. One row per channel. Adding a channel is a new row. |
| Pause a job                                                | Settings → the job's row → Mode = Paused                            |
| See errors                                                 | Pipeline Errors → Open                                              |
| Reconnect a social account                                 | The distributor's dashboard, signed in as the account's owner       |

## Next Manual Step

The formula numbers the human work by urgency. Blank means the row is not waiting on a person.

1. Fix failed posts
2. Review the content (and approve its posts, if any are waiting)
3. Approve posts
4. Record or film
5. Finish and approve designs, or finish assets
6. Finish the draft or script
7. Triage an idea
8. Set the CMS document live
9. Held (the series is on hold until {{HOLD_UNTIL}})

## Status

Content: Idea → Briefed → Drafting → Assets In Progress → Needs Review → Approved → Scheduled → Published. Archived and Rejected leave the queue.

- **Briefed** is the Copy Writer's cue. Within a few hours ({{COPY_SCHEDULE}}) it drafts one post per channel and moves the row to Needs Review.
- **Approved** is yours to set, as yourself. The scheduler moves it to Scheduled and Confirm Posts moves it to Published.

Ideas: New → Triage → Promoted, Parked, or Rejected. **Promoted** creates the Content row. Do not also create it by hand. An idea with Sensitivity Flag ticked will not promote until you have read it and set Redaction Status to Human-Checked.

## Approval

You approve by clicking in Content Review or the grid, signed in as yourself.

- A post is fully approved only when Approval is Approved **and** both Copy Approved and Creative Approved are ticked. Tick Creative Approved on a text-only post too: it confirms no creative is needed.
- Approved By and Approved At are stamped by an automation from your click. Never type them. A stamp an agent or a script set does not count: the distributor refuses it.
- Who may approve each channel is in Platform Rules → Allowed Approvers: {{APPROVERS}}.
- A personal account is approved only by its owner: {{PERSONAL_CHANNELS}}.
- **Changes Requested** clears the stamps and both boxes. Write what to change in Review Notes; the Copy Writer revises it at its next run and sends it back to Needs Review. Start the note with `CREATIVE:` when only the design needs work, so the Copy Writer leaves the copy alone.
- **Held series** ({{HELD_SERIES}}) do not go out even when every box is ticked.
- **Media needs its own approval.** A post with an image, carousel, thumbnail, or video goes out only when every file on it was approved by a person in Asset Review. Ticking Creative Approved on the post is not enough.

**Parking:** set Approval to Not Needed and start Review Notes with `PARKED <YYYY-MM-DD> (<your name>): <reason>`. To revive the post, set Approval back to Needs Review. Use `STALE` instead of `PARKED` for a repeat of a post that is already live.

## Scheduling

- The scheduler gives every fully approved post the channel's next free slot ({{SCHEDULER_INTERVAL}}-minute cycle), Priority content first.
- **Launch At** on the Content row pins every linked post to that time. It wins over the channel's slots.
- A time you type into Scheduled DateTime is never overwritten. If you change Launch At after posts were scheduled, clear their Scheduled DateTime and the scheduler refills them.
- A post more than {{MISSED_AFTER_MINUTES}} minutes late is marked Missed instead of going out late. Give it a new time and set Upload Status back to Pending.
- **Manual posts.** A post whose Delivery Method is Manual is never sent by a job. At its scheduled time, post it yourself (with the platform's AI label if Has Synthetic Media is ticked), paste the live URL into Platform Post URL, and set Upload Status to Posted. Analytics picks it up when its source can find the post by its URL; otherwise its metrics stay blank.

## Designs (creative module)

When you tick **Copy Approved** on a post that needs an image, carousel, or thumbnail (tick it as soon as the words are right; the post can stay in Needs Review), `CP · Creative Drafts` makes a draft design from that channel's brand template ({{CREATIVE_SCHEDULE}}) and attaches it to the post as a **Draft** asset with a preview and an edit link.

1. Open the asset in Asset Review, then the design. Finish it: images, layout, anything the template could not do.
2. Set the asset to **Ready for Review**.
3. An approver approves it with **Approve design**, signed in as themselves. That stamp is what lets the post go out. If the post itself was sent back or not yet approved, approve it too: both boxes, then Approval. {{#if FILE_HOST_WRITE}}The routine then saves the final file and writes its public link.{{/if}}{{#if FILE_HOST_MANUAL}}Then export the design into `{{MEDIA_ROOT}}/social/<content id>/` and run the media-links job.{{/if}}
4. Changed your mind after approving? Set the asset back to In Progress first. A design changed after its approval is never exported.
5. Want a fresh draft? Set the post to Changes Requested with Review Notes starting `CREATIVE: redraft`. Any other note starting `CREATIVE:` is posted on the design as a comment, and the copy is left alone.

No template for a channel and kind means no draft: design it by hand and add an asset yourself, or ask {{OWNER_NAME}} to add a template.

## Video and voice

Video and voice are always finished by a person, whatever tool prepared them. An editor's agent may rough-cut or caption, and a voice tool may draft a read; a person makes the final cut and approves every read before its asset is approved.

## Channels and connections

- Each channel's account sits on a profile at {{DISTRIBUTOR}}. If a platform disconnects, Distribute stops sending to it and writes one error row a day: its owner reconnects it in the distributor's dashboard. Nothing is sent on a guess.
- A post on a channel with Delivery Method = Manual is yours to post (see Scheduling).
- On X, links may be stripped by the distributor (see the X channel's Caption Style Notes). Put the link where the notes say.

## What you type vs what jobs type

| You type                                                                | Jobs type                                          |
| ----------------------------------------------------------------------- | -------------------------------------------------- |
| Brief, review notes, approvals, Launch At, Priority                     | Draft captions and hashtags (Copy Writer)          |
| Public Media URL on an Asset, after export (or run the media-links job) | Scheduled DateTime, when it was blank              |
| Asset Status: Ready for Review, Approved (Approve design)               | Draft designs and Draft assets (Creative Drafts)   |
| Clip Content Score, before a clip posts                                 | Upload Status, request id, live URL, Published At  |
| Platform Rules: times, days, approvers, caption notes                   | Views, likes, Engagement Rate, Engagement Score    |
| Use for Content on a meeting (opt-in)                                   | Ideas from opted-in meetings, Pipeline Errors rows |
| Manual posts: the live URL and Upload Status = Posted                   | Settings → Last Run Summary                        |

Never edit Engagement Score. Never type a live URL you did not copy from the platform.

## Scores

**Clip Content Score** (before posting): hook, standalone, relevance, brand safety, production, 0–2 each. A brand-safety problem caps it at 4. Under {{CLIP_SCORE_MIN}}, the clip goes back to review and the distributor will not send it.

**Engagement Score** (after posting): written daily, once a post has {{SCORE_MIN_VIEWS}}+ views and is {{SCORE_MIN_HOURS}} hours old. 5 means typical for us on that channel. If every scored post of a piece (at least two) scores {{BACK_TO_REVIEW_MAX_SCORE}} or less, its remaining approved posts come back to Needs Review with a `BACK TO REVIEW` note. Look at the angle before you approve them again.

## Media

- Files live under `{{MEDIA_ROOT}}` in {{FILE_HOST}}. A new export is a new file with a new version number; the Asset's Notes keep the old link.
- Public Media URL must be a direct download. The media-links job makes and checks them. Never paste an Airtable attachment link: it expires within hours.
- Repointing one Asset repoints every post that uses it.

## CMS

Rows for {{CMS_DOC_TYPES}} carry CMS Document ID, CMS Type, and CMS Live. A document is public only when it is published **and** live. Jobs create drafts with live off; a person publishes and sets it live in {{CMS}}. "8 · Set live in {{CMS}}" means that switch is still off. {{#if PROMOTE_NEW_POSTS}}New live posts appear in Content each night as Briefed rows, so the Copy Writer drafts posts that promote them.{{/if}}

## Meetings

Only meetings with **Use for Content** ticked are ever read. The Meeting Idea Miner removes names, organisations, money, personal matters, and anything said off the record, and writes generic ideas with Source Ref ids only. Ideas it flags as sensitive wait for you to read them and set Redaction Status to Human-Checked before they can be promoted.

## Pausing and modes

Each job has a row in Settings. Mode moves Paused → Dry Run → Drafts → Live. Only {{OWNER_NAME}} raises it. A guard may lower it (Distribute drops to Dry Run after {{ERROR_STREAK_GUARD}} send errors in one run) and writes the reason in Kill Reason. Read Kill Reason, fix the cause, then raise the Mode again.

## Errors

Every new Pipeline Errors row emails {{ALERT_EMAIL}}. Repeats of the same failure add to Occurrences instead of sending more mail. Triage: Investigating while you look, then Resolved or Ignored. Never put tokens, passwords, or meeting text in Error Message or Notes.

A post in **Error** is never resent by a job. Check the platform first (it may have gone out), fix the cause, then set Upload Status back to Pending.

## Weekly digest

{{DIGEST_DAY_TIME}}, to {{DIGEST_TO}}: posts published last week with their scores, open errors, the review queue, the next seven days, and held content.

## Changing the system

Change the brief first, then the system. Releasing a held series is one formula edit (Content → Is Held). Adding a channel is a Platform Rules row plus a matching option on Platform Posts → Platform. Changing the voice is a new brand pack version. Ask {{OWNER_NAME}} before anything else.
