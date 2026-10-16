# CP · Confirm Posts

| Setting | Value                                                                                                                                                                         |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trigger | At the end of each Distribute run (or on its own every 15 minutes). In the agent-routine variant, at the start of each `CP · Distribute` run.                                 |
| Reads   | Platform Posts in Sent; the distributor's status endpoint                                                                                                                     |
| Writes  | Platform Posts → Upload Status, Distributor Job Status, Platform Post URL, Published At, Error Details, Notes; Content → Status (Scheduled → Published only); Pipeline Errors |
| Modes   | Paused, Live                                                                                                                                                                  |

## Purpose

Turn a request id into a confirmed, live post with its real URL, or into an error a person can see. Distribution is asynchronous: the send returns a request id, and the outcome arrives later.

## Rules

- **Platform Post URL only from the distributor's response.** Never build one from a pattern, never guess.
- Posts with Delivery Method = Manual are a person's: never touch them.
- **Never resend anything,** and never call the distributor's retry tool. A stuck or partial post goes to Error for a person to check on the platform first: resending blind can post twice.
- One status call per post per run. Do not poll in a loop inside one run; the next run asks again.

## Steps

1. Read the Settings row. Paused: stop.
2. Find posts with Upload Status = Sent. Batch: up to `{{CONFIRM_BATCH | 50}}`, oldest Sent At first.
3. For each:
   - **No Distributor Request ID** and Sent At more than 30 minutes ago: Upload Status = Error, Error Details `Sent without a request id: check the channel before resending`.
   - Otherwise ask the distributor for the request's status (Upload-Post: `get_status` with the `request_id`; a workflow calls the same status endpoint). Read the result for the post's one platform:
     - **Success** (Upload-Post: `status` is `success` and the platform's result succeeded): Upload Status = Posted, Distributor Job Status = Completed, Platform Post URL from the platform's result, Published At from the response (unwrap `{"$date": …}`; convert to UTC). No URL in the response: leave the field empty and add the Notes line `posted, no URL returned`. Never type one.
     - **Failed:** Upload Status = Error, Distributor Job Status = Failed, Error Details = the scrubbed reason.
     - **Partial** (the request reports success on some targets and failure on others, or success with the platform's own result missing): Upload Status = Error, Distributor Job Status = Partial, Error Details `Partial result: check the channel before resending`.
     - **Still processing:** leave it. If Sent At is more than `{{CONFIRM_TIMEOUT_HOURS | 6}}` hours ago: Upload Status = Error, Error Details `No confirmation after <n> h: check the channel before resending`.
   - **Warnings.** The response may carry warnings on a post that succeeded (Upload-Post: `warnings`). Write each one, scrubbed, as a Notes line. A warning that the platform removed something the post relied on (on X, links are stripped from posts and first comments unless the account has the distributor's link add-on; an optional field the connection does not support) also creates one Pipeline Errors row, so a person fixes the channel's Caption Style Notes.
4. For each Content row touched: if every linked post with Approval = Approved now has Upload Status = Posted, and Content Status is Scheduled, set Content Status = Published.
5. Write the Last Run Summary: posted, failed, partial, still processing, warnings, record ids.

## n8n notes

- Upload-Post: the status endpoint takes the `request_id` from the upload response. Run it as a second branch at the end of Distribute, or as its own workflow on the same 15-minute schedule.
- The status of a scheduled or queued upload is a different call (`get_job_status` with a `job_id`). This kit never schedules inside the distributor, so it never needs it. A `job_id` in a response means something scheduled inside the distributor: raise an error row.

## Test

After the Distribute test post goes out, the next run sets Posted, fills the URL, and the URL opens the live post. Set a `[TEST]` post to Sent with no request id and a Sent At an hour ago: it goes to Error with the "check the channel" note. Delete the test post and say so.
