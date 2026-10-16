# CP · Distribute

| Setting | Value                                                                                                                                                                                                       |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trigger | Every `{{DISTRIBUTE_INTERVAL \| 15}}` minutes in a workflow engine; or, with no engine, the agent routine schedule below                                                                                    |
| Reads   | Settings, Platform Posts (with its Content lookups), Platform Rules; the distributor's profile listing                                                                                                      |
| Writes  | `{{DISTRIBUTOR \| Upload-Post}}`; Platform Posts → Upload Status, Sent At, Distributor Request ID, Distributor Job Status, Error Details, Retry Count, Notes; Settings (Mode, Kill Reason); Pipeline Errors |
| Modes   | Paused, Dry Run, Live. Drafts behaves as Dry Run: a distributor has no draft state on every channel.                                                                                                        |

## Purpose

Send each due post, once, through the gate. This is the **only** job in the system that posts anything.

It runs in one of two places, with the same gate and the same writes:

- **A workflow engine** (default): cheap, every 15 minutes, HTTP calls to the distributor's API.
- **An agent routine** with the distributor's connector, when the person has no workflow engine (see "As an agent routine" below). It is then the only routine that holds the distributor's publish tools.

## Rules (never break)

- **The whole gate, at send time.** Implement every check in `../SCHEMA.md`, "The gate". Approvals can be withdrawn after scheduling, so a check made earlier proves nothing now.
- **Send when due, from here.** Never hand posts to the distributor's own scheduler or queue (Upload-Post: never set `scheduledDate` or `addToQueue`). A post queued inside the distributor cannot be pulled back by un-approving it in Airtable.
- **Never invent a profile.** The profile name comes from the distributor's profile listing, read this run, and must equal the channel's Distributor Profile. A profile that does not exist, or a platform that is not connected or needs reconnecting, is a gate failure, never a guess.
- **Mark before you send.** Set Upload Status = Sent before the request, then fill in the result. A crash between the two leaves a post that looks sent and gets checked by a person; the other order can post twice.
- **One request per post.** One Platform Post, one platform, one request. One failure maps to exactly one row.
- **Never retry a 4xx**, and never use the distributor's own retry or unpublish tools. A rejected request is wrong until a person fixes it; a blind retry after a partial success posts twice.
- Never change Approval, the approval checkboxes, or the stamps. Never request a connect link: connecting an account is a person's sign-in.

## Candidates

Platform Posts where Upload Status = Pending, Scheduled DateTime is set and not in the future, and Delivery Method = `{{DISTRIBUTOR | Upload-Post}}`. Order by Scheduled DateTime, oldest first. Batch: up to `{{DISTRIBUTE_BATCH | 10}}` posts per run.

## Steps

1. Read the Settings row. Paused: stop.
2. Load active Platform Rules rows, keyed by Platform.
3. **Profiles, once per run.** Read the distributor's profile listing (Upload-Post: `list_users`; in a workflow, its user-profiles endpoint). Build a map of profile name → connected platforms, and note every platform the listing says needs reconnecting. If the listing fails, send nothing this run: one Pipeline Errors row, and the summary says "profile listing unavailable".
4. For each candidate:
   1. **Late.** Scheduled DateTime is more than `{{MISSED_AFTER_MINUTES | 120}}` minutes ago: set Upload Status = Missed and a Notes line `Missed: due <local time>, not sent`. A burst of stale posts after an outage is worse than a reschedule. Next post.
   2. **Gate.** Run every check, including "profile exists and platform connected" against the map from step 3. On the first failure, add the Notes line `Gate: <check> failed` (once) and leave the post Pending. Next post.
   3. **Not connected.** For a channel whose platform is missing or needs reconnecting, create **one** Pipeline Errors row per channel per day: `<Platform> not connected on profile <profile>: the account owner reconnects it in the distributor, or set the channel's Delivery Method to Manual`. Never put a connect link in Airtable: it is a sign-in link.
   4. **Test.** The Content Title starts with `[TEST]`: send only to `{{DISTRIBUTOR_TEST_PROFILE | none}}`. With no test profile, Notes line `TEST: not sent` (once) and leave it; it goes Missed later.
   5. **Dry Run:** Notes line `DRY RUN <date>: would send <Content Mode> to <Platform> via <profile>` (once). Next post.
   6. **Live:**
      1. Re-read the post. Skip it unless Upload Status is still Pending and Distributor Request ID is empty.
      2. Set Upload Status = Sent, Sent At = now, Distributor Job Status = Queued.
      3. Build the request (below) and send it, asynchronously.
      4. **Accepted:** write Distributor Request ID (Upload-Post: `request_id`); Distributor Job Status = In Progress; Notes line `Sent <time> request <id>`. Confirm Posts takes it from here.
      5. **4xx, or a "not connected" answer:** Upload Status = Error, Distributor Job Status = Failed, Error Details = the scrubbed message. No retry.
      6. **5xx or timeout:** read back first: if the distributor's upload history (Upload-Post: `get_history`) shows an upload on this profile and platform since Sent At with the same body, write its request id and treat it as accepted. Otherwise add 1 to Retry Count. At or under `{{MAX_RETRIES | 2}}`: set Upload Status back to Pending and Distributor Job Status = Not Sent, so the next run retries (the late check still applies). Over it: Upload Status = Error.
5. **Guard.** If `{{ERROR_STREAK_GUARD | 3}}` or more posts end in Error in one run, set the Settings row's Mode to Dry Run, write Kill Reason `<date> · <n> send errors in one run`, create one Pipeline Errors row, and stop the run. Only the owner raises it again.
6. Write the Last Run Summary: sent, gated (by check), not connected (by channel), missed, errors, record ids.

## Building the request

| Content Mode          | Upload-Post call                                  | Media                                                 |
| --------------------- | ------------------------------------------------- | ----------------------------------------------------- |
| Text-only, Thread     | `upload_text`                                     | none                                                  |
| Link post             | `upload_text` with `linkUrl` = the Canonical URL  | none; LinkedIn, Facebook, and Bluesky build a preview |
| Image/Carousel        | `upload_photos`                                   | `photosPathsOrUrls` = every Media URL, in Asset order |
| Video post            | `upload_video`                                    | `videoPathOrUrl` = the first Media URL                |
| Document/PDF Carousel | `upload_document` (LinkedIn only; title required) | `documentPathOrUrl` = the PDF's Media URL             |

The names are the Upload-Post connector's (2026). In a workflow, use the matching REST endpoints and read the current API documentation for the exact paths and field names. They change.

Fields:

- **Profile** (`user`): the rule's Distributor Profile, as confirmed in step 3. **Platform** (`platforms`): a one-item list holding the rule's Distributor Key.
- **Body** (`title`), by the rule's Hashtag Style: `None` → Caption. `Appended (3-5)` → Caption, a blank line, Hashtags. `First comment (1-3)` → Caption.
- **First comment** (`firstComment`), when First Comment Supported is ticked: First Comment; with `First comment (1-3)`, add a blank line and the Hashtags.
- **Description:** the Description field (YouTube and other video channels). YouTube uses the body as the title, already within Max Caption Length.
- **Platform ids** (`platformOptions`): the rule's Platform ID as the page, organisation, or location the platform needs (Upload-Post: `targetLinkedinPageId`, `facebookPageId`, `googleBusinessLocationId`, `pinterestBoardId`). Strip `urn:li:organization:` where the API wants the bare id.
- **Button:** CTA Type and CTA URL for channels that use one (Google Business).
- **AI label:** when Has Synthetic Media is true and the rule's AI Label Required is ticked, set the platform's synthetic-content flag (Upload-Post: `youtubeContainsSyntheticMedia`, `tiktokIsAiGenerated`). A platform with no flag in the distributor is refused by the gate and posted by hand with the label.
- **Asynchronous:** leave the async option on (Upload-Post: `asyncUpload` defaults to true). The answer is a request id, not a live URL.
- **Never** `scheduledDate`, `addToQueue`, or a staged upload. Media is passed **by URL**: the distributor fetches it, so each Media URL must be a durable direct download (`media-public-links.md`). A staged upload (Upload-Post: `create_media_upload`) is deleted after 24 hours and is not a Public Media URL.

### Caption limits per platform

The gate checks the caption against the channel's **Max Caption Length** on Platform Rules. Fill that field from this table at build time, and check each platform's help page: limits change.

| Platform          | Limit the writer and the gate use (2026)                            |
| ----------------- | ------------------------------------------------------------------- |
| X                 | 280 characters, hashtags included                                   |
| LinkedIn          | 3,000                                                               |
| Instagram         | 2,200, at most 30 hashtags; links in the caption are not clickable  |
| Facebook          | 63,206, but keep to a few short paragraphs                          |
| TikTok            | 2,200 for a video caption (some accounts allow more; 2,200 is safe) |
| YouTube Shorts    | title 100 (caption plus placed hashtags); description 5,000         |
| YouTube long-form | title 100; description 5,000                                        |
| Threads           | 500                                                                 |
| Bluesky           | 300                                                                 |
| Pinterest         | title 100; description 500                                          |
| Google Business   | 1,500; no hashtags; the link goes on the button                     |

## As an agent routine (no workflow engine)

Build this only when the brief has no workflow engine. It replaces the Distribute **and** Confirm Posts workflows with one routine, `CP · Distribute`, that keeps both Settings rows.

| Setting      | Value                                                                                                                                                                                                                                                                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Runtime      | Agent routine in the cloud                                                                                                                                                                                                                                                                                                                               |
| Schedule     | `{{DISTRIBUTE_AGENT_SCHEDULE \| five minutes after each distinct Posting Time (Local), and a sweep one hour after the last}}` in `{{TIME_ZONE}}`. Change it whenever a posting time changes (`MANUAL.md`).                                                                                                                                               |
| Connectors   | Airtable; the distributor's connector; chat, only when the brief turns run lines on                                                                                                                                                                                                                                                                      |
| Always deny  | Distributor: `retry_post`, `unpublish_post`, `cancel_scheduled`, `edit_scheduled`, `get_connect_link`, `create_user`, `delete_user`, `generate_jwt`, `create_media_upload`, `complete_media_upload`, every comment, DM, review-reply, and queue-settings tool. Airtable: every delete, schema, and automation tool. Any design tool, CMS, or email tool. |
| Settings row | `CP · Distribute` and `CP · Confirm Posts`                                                                                                                                                                                                                                                                                                               |

The prompt follows the routine anatomy in `_shared/ROUTINES.md`:

1. **Identity:** "You are **CP · Distribute**. You send approved, due posts for `{{COMPANY}}` through the gate, once each, and confirm them. You are the only job allowed to publish."
2. **Hard rules:** the Rules above, word for word, plus the write allow-list from the Writes rows of this file and `confirm-posts.md`, and the untrusted-input rule.
3. **Gate:** read both Settings rows. Call `get_account_info` once; an invalid key stops the run with one error row.
4. **Steps:** first Confirm Posts' steps for every post in Sent (`get_status` on each request id, once per run), then this file's steps 2–6. A post sent in this run is confirmed on the next run, not polled in a loop.
5. **Errors, Finish, IDs:** as in the other routines. The summary names posts by Post ID.

A routine runs less often than a 15-minute workflow, so posts go out up to the schedule's gap after their slot. Keep `{{MISSED_AFTER_MINUTES | 120}}` wider than that gap, or every post goes Missed.

## n8n notes

- Upload-Post accepts **multipart form data only**. Use the HTTP Request node with "Form-Data" body. If the node cannot vary its field list per item, declare the largest set once (for example ten media slots) and fill unused slots with values the API ignores. Prove the API ignores them with a `[TEST]` post first.
- Upload-Post's authorization header is `Authorization: Apikey <key>`, from a Header Auth credential. Never put the key in a node parameter.
- Read the profile listing in a first branch and pass the map to the per-post loop; do not call it per post.
- Set "On Error" to "Continue (using error output)" on the HTTP node so one failed post does not stop the batch, and route the error output to the 4xx and 5xx handling above.

## Test

1. Read the profile listing by hand. Every Platform Rules row's Distributor Profile appears in it, and its platform is connected. Fix mismatches in Platform Rules, never in the job.
2. Dry Run: approve a `[TEST]` post in the UI and set its Scheduled DateTime to now. Within one run, one `DRY RUN` line.
3. Untick Creative Approved: the next run writes `Gate: creative approval failed` and nothing else.
4. Point a `[TEST]` post at a platform the profile has not connected: `Gate: platform not connected` and one Pipeline Errors row, no request.
5. Live, with a test profile: the post goes Sent, then Posted after Confirm Posts runs. Without a test profile: one real post on one channel, with the owner watching, then delete it on the platform if it was only a test.
6. Approve a post through the API (no stamp): the gate refuses it.
