# CP · Error Handler

| Setting | Value                                                         |
| ------- | ------------------------------------------------------------- |
| Trigger | Any other `CP ·` workflow fails (n8n: Error Trigger node)     |
| Reads   | The failed execution's metadata                               |
| Writes  | Pipeline Errors (create, or update Occurrences and Notes)     |
| Modes   | None. It always runs, even when its Settings row says Paused. |

## Purpose

Turn every failed run into one row a person can act on. The Airtable automation "Pipeline Errors: email on new row" emails `{{ALERT_EMAIL | the owner}}`. The handler itself sends nothing.

## Rules

- Never write secrets: strip `Authorization` values, `Bearer …`, `Apikey …`, `key=`, `token=`, `secret=`, `password=` (and their values) from every string, and anything that looks like a long random key.
- Never write request bodies, captions, transcripts, or message text. Node name, error message, and record ids are enough.
- Truncate Error Message to 2,000 characters.
- **Dedupe.** A job that runs every 15 minutes can fail 96 times a day. If a row with the same Workflow, Failed Node, and the same first 200 characters of the message exists with Status New or Investigating and Occurred At in the last 24 hours, add 1 to its Occurrences and add one dated Notes line. Do not create a new row: one email per problem, not per run.

## Steps

1. Read the execution: workflow name and id, execution id, the last node that ran, the error message, the time.
2. Build the execution URL: `<engine base URL>/workflow/<workflow id>/executions/<execution id>` for n8n.
3. Scrub and truncate the message.
4. Pull record refs if the failed item carried them (a Post ID, a Content ID, an Airtable record id). Never the item's other fields.
5. Search Pipeline Errors for a duplicate (Rules, "Dedupe").
6. Duplicate: update Occurrences and Notes. Otherwise create a row: Summary = `<workflow>: <node> failed` (one line), Source "Workflow", Workflow, Workflow ID, Execution ID, Execution URL, Failed Node, Error Message, Record Refs, Occurrences = 1, Occurred At = the failure time, Status "New".
7. If writing to Airtable itself fails (Airtable is down), send one message to `{{OWNER_CHAT_TARGET | the owner's DM}}` with the workflow name and execution URL, and nothing else.

## n8n notes

- Build it with an **Error Trigger** node. Set it as the error workflow in every other `CP ·` workflow's settings.
- The Error Trigger only fires for workflows that are active and failed in production runs. Test it by activating a throwaway workflow that throws, then deleting that workflow.

## Test

Activate a `[TEST]` workflow that throws. One row appears with a scrubbed message. Trigger it twice more within a minute: still one row, Occurrences = 3. The email arrives once. Set the row to Ignored and delete the test workflow.
