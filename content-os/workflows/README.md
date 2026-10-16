# Content OS — workflows

The deterministic jobs: polling, API calls, retries. They run in a workflow engine because they need no judgement and run often. Each file is a spec written for any engine, with notes for n8n (the default). Make, Zapier, and Pipedream can run every one of them.

Without an engine, an agent routine runs the same spec through the tools' connectors. It costs more per run and runs less often, so the kit merges Distribute and Confirm Posts into one routine (`distribute.md`, "As an agent routine") and keeps every other rule. The gate is the same either way. The design step is not here: it needs judgement, so it is a routine (`../routines/creative-drafts.md`).

## The jobs

| Job                                | File                    | Cadence                                       | Reads                                                      | Writes                                                            | Modes                 |
| ---------------------------------- | ----------------------- | --------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------- | --------------------- |
| `CP · Error Handler`               | `error-handler.md`      | On any other job's error                      | the failed execution                                       | Pipeline Errors                                                   | always on             |
| `CP · CMS Inventory Sync`          | `cms-inventory-sync.md` | Nightly, `{{INVENTORY_SYNC_TIME \| 02:00}}`   | the CMS (read-only)                                        | Content (CMS fields; new rows for live documents)                 | Paused, Live          |
| `CP · Scheduler`                   | `scheduler.md`          | Every `{{SCHEDULER_INTERVAL \| 30}}` minutes  | Platform Posts, Platform Rules                             | Scheduled DateTime, Notes; Content Status Approved → Scheduled    | Paused, Dry Run, Live |
| `CP · Distribute`                  | `distribute.md`         | Every `{{DISTRIBUTE_INTERVAL \| 15}}` minutes | Platform Posts, Platform Rules, the distributor's profiles | the distributor; Upload Status, request id, Notes                 | Paused, Dry Run, Live |
| `CP · Confirm Posts`               | `confirm-posts.md`      | After each Distribute run                     | the distributor's status endpoint                          | Upload Status, live URL, Published At; Content Status → Published | Paused, Live          |
| `CP · Analytics Pull`              | `analytics-pull.md`     | Daily, `{{ANALYTICS_TIME \| 06:00}}`          | the distributor's or platforms' analytics                  | metrics, Engagement Rate and Score; back-to-review moves          | Paused, Dry Run, Live |
| `CP · One-off: Media Public Links` | `media-public-links.md` | Manual, per batch                             | the file host                                              | Assets → Public Media URL; page assets of approved designs        | manual                |

Times are in `{{TIME_ZONE}}`.

## Rules every job keeps

- **Read the Settings row first.** Look up the row whose Job equals the workflow's name. Paused: stop. Dry Run: write only `DRY RUN` lines. Live: act. The Error Handler is the exception: it always runs.
- **Write allow-list.** Each spec names the fields it writes. Nothing else. Never delete a record.
- **Address fields by id**, from the brief's ID map. Names drift; ids do not.
- **Idempotent.** Re-read a record right before writing it. If it changed since the search (a person set a time, a post was already sent), skip it.
- **One dated line, once.** A job that explains itself in a Notes field writes `YYYY-MM-DD HH:mm · <job>: <what>` on top, and skips the write when the newest line already says the same. Jobs that run every 15 minutes otherwise bury the field.
- **Batch limits.** Each spec names one. The rest waits for the next run.
- **Never retry an item twice in one run.**
- **Time.** Convert every UTC value to `{{TIME_ZONE}}` before comparing it with a local time, and back to UTC before writing a date-time. Daylight-saving changes move the offset; compute it per date.
- **Secrets** live in the engine's credential store, created by a person. A job never logs headers, tokens, or full request bodies.
- **Test mode.** A Content Title starting `[TEST]` never reaches a real audience.
- **Last Run Summary.** Each run writes one line to its Settings row: counts and record ids only.
- **Error workflow.** Every job names `CP · Error Handler` as its error workflow.

## The gate

`distribute.md` re-checks the whole gate in `../SCHEMA.md` at send time. The scheduler checks the approval half before it assigns a slot, so it does not fill the calendar with posts that will be refused. Neither job relaxes a check, and no other job sends anything.

## Gotchas found in production

| Area              | Gotcha                                                                                                                                                                                                                                                                  |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Distribution APIs | Some accept **multipart form data only** and reject JSON (Upload-Post does). If the engine's HTTP node needs a fixed list of form fields, declare the full set once and fill unused slots with values the API is proven to ignore, or build the form in a code step.    |
| Distribution APIs | Date fields can come back wrapped, for example `{"$date": "…"}` (Upload-Post's `upload_timestamp`). Unwrap before writing to Airtable.                                                                                                                                  |
| Distribution APIs | Uploads are asynchronous: the first response is a request id, not a live URL. Poll the status endpoint (Confirm Posts). Never type a URL you did not receive.                                                                                                           |
| Distribution APIs | Send one request per Platform Post, so one failure maps to exactly one row.                                                                                                                                                                                             |
| Distribution APIs | Read the profile listing (Upload-Post: `list_users`) before sending. It names each profile exactly and says which platforms are connected or need reconnecting. A post to an unconnected platform fails; a guessed profile name posts nowhere, or to the wrong account. |
| Distribution APIs | A connect link (Upload-Post: `get_connect_link`) is a sign-in link for a person, valid for a limited time. On an account with no profile it also **creates** one. Never call it from a job; the build agent asks first, and hands the link to the account's owner.      |
| Distribution APIs | Upload-Post removes links from X posts and first comments unless the account has its X link add-on. Without it, the X caption carries no URL (say so in the X channel's Caption Style Notes). The status response warns when it happens.                                |
| Distribution APIs | Staged uploads (Upload-Post: `create_media_upload`) are deleted after 24 hours. They are for one immediate send, never a Public Media URL.                                                                                                                              |
| Distribution APIs | A request can succeed on the distributor and still carry per-platform warnings or a partial result. Read the platform's own entry in the status response, not only the top-level status.                                                                                |
| Design tools      | Export links (Canva: `export-design`) expire. Attach the export to Airtable as a preview (Airtable copies the file when the attachment is written) and make the durable file through the file host.                                                                     |
| Design tools      | Brand kits, brand templates, and template autofill depend on the design tool's plan. Discovery shows what this account has. An empty brand-kit list means the wrong account or a plan without kits, never "no brand".                                                   |
| Platform ids      | Store LinkedIn organisations as `urn:li:organization:<id>` and strip the prefix where an API wants the bare id.                                                                                                                                                         |
| Caption limits    | YouTube Shorts uses the caption plus hashtags as the title, at most 100 characters. X allows 280; put hashtags in the first comment. Google Business takes no hashtags and carries its link on a button.                                                                |
| n8n + Airtable    | The Airtable search node nests values under `json.fields`. A Code node that reads `json.Caption` gets `undefined`.                                                                                                                                                      |
| n8n + Airtable    | Turn the node's typecast option off. Typecast on a select silently creates a misspelled option.                                                                                                                                                                         |
| n8n               | Runs started through the n8n MCP need `executionMode: "manual"`.                                                                                                                                                                                                        |
| n8n               | A Schedule Trigger fires in the workflow's time zone setting, which defaults to the instance's. Set it to `{{TIME_ZONE}}` on every workflow.                                                                                                                            |
| n8n               | The built-in Dropbox credential cannot create shared links. Create a generic OAuth2 credential with the `sharing.write` scope for the media-links job.                                                                                                                  |
| Airtable          | The API reads and writes date-times in UTC whatever the field's display zone. A local `HH:mm` text field and a UTC date-time field side by side invite a scheduler to read the wrong one.                                                                               |
| Airtable          | Attachment URLs expire within hours. Never hand one to a distributor or store one as a Public Media URL.                                                                                                                                                                |
| Airtable          | About 5 requests per second per base. Batch writes (10 records per call) and page reads.                                                                                                                                                                                |
| File hosts        | A `HEAD` request on a Dropbox direct link answers `application/json`. Check a media link with a ranged `GET` (`Range: bytes=0-1023`): expect `206` or `200` and a media content type.                                                                                   |
| File hosts        | Cloud agent sandboxes block direct-download domains (Dropbox: `*.dl.dropboxusercontent.com`) until the environment allowlists them.                                                                                                                                     |
| File hosts        | Google Drive direct links return an HTML warning page for large files instead of the file. The ranged GET catches it; host large video elsewhere.                                                                                                                       |
| File hosts        | S3 presigned URLs expire. Store a public object or CDN URL, never a presigned one.                                                                                                                                                                                      |
| Audio             | Social audio targets `{{LOUDNESS_TARGET \| -14 LUFS integrated, true peak at or below -1 dBTP}}`. AAC encoding adds about 0.5 dB of true peak, so limit before you encode.                                                                                              |
| Video             | `ffmpeg -shortest` cuts the video to the audio's length. Pad the audio instead. Export at 30 fps with captions burned in or attached: most feeds play muted.                                                                                                            |
| Editors           | Descript: record voiceover on its own layer, not the script track (recording there shifts the timeline). Transcript export covers the script track only. A publish adds about 2 seconds of tail.                                                                        |

## Make, Zapier, Pipedream

- A **scenario** (Make), **Zap** (Zapier), or **workflow** (Pipedream) per job, in one folder named like the n8n folder.
- Schedules: Make and Pipedream support 15-minute intervals on paid plans; check Zapier's plan minimum before promising 15 minutes.
- Error handling: Make's error-handler route, Zapier's error paths, or Pipedream's `$.flow.exit` plus an error step that writes the Pipeline Errors row.
- Multipart uploads need a raw HTTP module that supports form data with files by URL.
