# Blog writing — operator's manual

> **Build agent.** Fill every `{{…}}` from the brief and the ID map, remove the `{{#if}}` markers, delete this note, and hand the result to the team. Keep it short enough to read on a phone.

Live base: [{{BASE_NAME}}]({{BASE_URL}}). Interface: [Blog Desk]({{INTERFACE_URL}}). Brief: [{{COMPANY}} blog writing brief]({{BRIEF_LOCATION}}).

Search tickets by **{{ID_PREFIX}}-B####**. Do not ask anyone for a record id.

## The week

| When                        | What happens                                                                                                          | Who                                 |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| {{SCAN_SCHEDULE_TEXT}}      | `BL · Content Gap Scan` writes up to {{WEEKLY_TARGET \| 7}} Proposed tickets and posts a digest in {{REPORT_CHANNEL}} | routine                             |
| Within a day                | Triage: approve (Ready), reject, or park each topic                                                                   | {{TOPIC_APPROVER}}                  |
| {{WRITER_SCHEDULE_TEXT}}    | `BL · Post Writer` drafts up to {{WRITER_BATCH \| 2}} Ready tickets and messages {{REVIEWER_CHANNEL}}                 | routine                             |
| Within a week of each draft | Review, publish, switch live on                                                                                       | {{REVIEWER_DEFAULT}}, {{PUBLISHER}} |
| {{REFRESH_SCHEDULE_TEXT}}   | `BL · Refresh Review` proposes refresh and correction tickets for posts due for review                                | routine                             |

## Where to click

| You want to                               | Open                                                    |
| ----------------------------------------- | ------------------------------------------------------- |
| See what is waiting on you                | Next Manual Step on any ticket, or Blog Desk → Pipeline |
| Approve or reject this week's topics      | Blog Desk → Triage                                      |
| Review drafts                             | Blog Desk → Review                                      |
| Fix a stuck ticket                        | Blog Desk → Blocked                                     |
| See refresh and correction work           | Blog Backlog → Refresh and Corrections                  |
| See what is live                          | Blog Backlog → Live Posts                               |
| See what a keyword is worth               | Blog Desk → Keywords                                    |
| See what a routine did, or why it stopped | Blog Desk → Runs                                        |
| Change modes, thresholds, or budgets      | Blog Desk → Settings (owner only)                       |

## Next Manual Step

Blank means the ticket is not waiting on a person.

1. Unblock: read Agent Notes, fix the cause, set Ready.
2. Review the draft, publish, switch live on.
3. Set Ready yourself in the UI so it is stamped.
4. Approve or reject the topic.

## Status

Proposed → Ready → Writing → Draft Ready → Live.

- **Changes Requested** sends a draft back to the writer (at most twice).
- **Blocked** means a guard stopped the writer. Agent Notes says why.
- **Parked** keeps a topic for later. **Rejected** keeps its keyword out of new tickets for {{REJECT_COOLDOWN_DAYS | 90}} days.

## Triage a topic

Open Blog Desk → Triage. For each Proposed ticket:

- Is the keyword real? Every number has a tool and a date. The SERP note says whether a site your size can rank.
- Is it for an audience you serve, in a cluster you chose, and something you would say?
- Geo: does it list at least {{GEO_MIN_FACTS | 3}} local facts with sources?
- Refresh: does it say exactly what changes, with the URLs that show it?

Then:

- **Approve:** set Status to Ready yourself, in the UI. The stamp is what lets the writer take it.{{#if AUTO_READY}} (Auto-Ready Topics is on, so the writer may take Proposed tickets anyway; approving still sets the order.){{/if}}
- **Reject:** set Rejected and write one line in Review Notes saying why. The scan learns nothing from silence.
- **Park:** set Parked.

You may edit Title, Priority, Category, CTA, and the outline before you approve. Do not edit Primary Keyword unless it is wrong; if it is, fix the field and the `Primary keyword:` line in the Brief together.

## Review a draft

Open Blog Desk → Review, then the Draft URL. Check against the ticket's ACCEPTANCE list:

- The answer box answers the title in 40–60 words, conclusion first.
- Every number matches its source. Open the sources that carry the post.
- No number from the never-type list: {{NEVER_TYPE_NUMBERS}}.
- At least three internal links, and they go where they say.
- The CTA is the right one for this reader.
- It sounds like {{COMPANY}} (the brand pack).

Want changes? Set **Changes Requested** and write what to change in Review Notes. The writer revises on its next run, unless you have already edited the draft in the CMS; then finish it yourself.

## Publish (two gates)

1. In the CMS: set the publish date; confirm category, author, reviewer, review date; add a cover if Agent Notes says `needs cover`.
2. Publish.
3. Open the preview and check it reads right.
4. Switch **live** on. Published alone is not public, and a forgotten switch is a silent 404. Switch on any supporting document the reviewer message lists too.
5. Airtable catches up on the writer's next run (Status Live, Live URL). You may set Live yourself.

Refresh: set the review date to today when you publish. Correction: check the dated entry shows in the sources and corrections block. News is corrected, never rewritten.

## Blocked tickets

| Agent Notes says                               | Do this                                                                                                                                       |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `slug taken`                                   | Pick a different Target Slug (a new URL is your decision), or turn the ticket into a Refresh of the post that holds the slug. Then set Ready. |
| `category missing`, `author missing`           | Create it in the CMS and publish it, or change the ticket's Category. Then set Ready.                                                         |
| `live count: "…"`                              | Reword that part of the outline in words that stay true. Then set Ready.                                                                      |
| `unsourced: …`                                 | Add a source to SOURCE NOTES, or cut that section from the outline. Then set Ready.                                                           |
| `draft discarded in CMS`                       | Set Ready to have it rewritten, or Rejected.                                                                                                  |
| `edited in the CMS` / `a draft already exists` | Finish the draft in the CMS yourself, then publish.                                                                                           |
| `two revisions`                                | Finish it in the CMS.                                                                                                                         |
| `ticket incomplete`                            | Fill the `<…>` blanks in the Brief. Then set Ready.                                                                                           |
| `schema mismatch`                              | Tell the owner. The CMS changed; the routines need refilling.                                                                                 |
| `EMBED PENDING: …`                             | The embed could not be built or verified (wrong account, empty result). Fix the account or the filters, then set Ready.                       |

## Refresh and corrections

| Ticket                         | Means                                                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| Refresh, Confirm               | Nothing found to change. Read the evidence, approve, and when the writer's draft arrives, publish it to re-date the post. |
| Refresh, Update                | A figure, rule, programme, or source changed. The draft changes only those parts.                                         |
| Refresh, Expand                | The post nearly ranks for more. The draft adds coverage.                                                                  |
| Correction                     | A statement is wrong. The draft fixes that sentence and adds a dated correction.                                          |
| `Owner decision:` in the title | The post's keyword has collapsed. Read the three options in Evidence and choose. Slugs never change without a redirect.   |

## Keyword Ledger

Every number the engine has paid for, so it is not paid for twice.

- **Dead Ends** (`Zero-volume verified`) never come back. Only you can revive one: set it to Candidate.
- **Rejected** shows why each keyword failed. Set one to Candidate to have the next scan look again.
- Never type a number into the ledger yourself unless you write its tool and date beside it (Tool `Other`, the source in Agent Notes).

## Settings (owner only)

- **Modes:** Gap Scan, Writer, Refresh. `Paused → Dry Run → Live`. Only the owner raises a mode. A guard may lower one and write the Kill Reason.
- **Thresholds and budgets:** change the brief first, then the row. Min Volume {{MIN_VOLUME | 30}}, Max Difficulty {{MAX_DIFFICULTY | 35}} (own authority {{DOMAIN_AUTHORITY}}, {{DA_PULLED}}), scan budget {{SCAN_CALL_BUDGET | 40}} calls, refresh budget {{REFRESH_CALL_BUDGET | 20}} calls.
- **Auto-Ready Topics:** on lets the writer skip topic approval. Publishing stays human either way.
- **Writer Min Gap Hours:** a run that comes too soon after one that drafted only reconciles. Set it to 0 only while testing.
- To stop everything at once, set all three modes to Paused.

## What you type and what jobs type

| You type                                                     | Jobs type                                                 |
| ------------------------------------------------------------ | --------------------------------------------------------- |
| Status changes to Ready, Rejected, Parked, Changes Requested | Proposed tickets, Writing, Draft Ready, Live, Blocked     |
| Review Notes                                                 | Agent Notes (append-only; never edit or delete a line)    |
| Priority, Title, Category, CTA before approval               | Keyword data, Evidence, Draft URL, Preview Path, Live URL |
| Settings                                                     | Run Log rows, last-run summaries                          |
| Tickets of your own (Source: Person)                         | Ledger rows                                               |

## Rules that never bend

- Nothing is switched live by a routine, and no post is published by one. {{#if SUPPORT_DOCS}}The writer may create supporting documents (glossary terms and the like) published with live off; you switch them on.{{/if}}
- Every keyword number carries its tool and date. Every statistic in a post has a source someone opened.
- Numbers on the never-type list never appear in a post.
- Slugs never change. A live URL is never silently retargeted.
- One live post per keyword: a repeat becomes a refresh.
- No keys or passwords in Airtable, Review Notes, or chat.

## Errors and budget

- Errors arrive by email and in Blog Desk → Runs → Errors. Move each one to Investigating, then Resolved or Ignored. Never paste a token, a header, or page text into a note.
- Each routine's summary shows the keyword calls it spent and the quota left. If the scan keeps stopping for quota, lower its budget or move its schedule; do not raise the budget past the plan.
- {{#if CONTENT_OS}}Live posts reach the Content OS through its nightly CMS inventory sync, which makes the Content row for social posts. The ticket's Content Ref links to it a run or two later.{{/if}}
