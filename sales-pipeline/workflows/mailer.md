# Workflow: the mailer

A small webhook that creates email drafts (and, only when switched on, sends) **as the sender, from the sender's real mailbox**, with clean links and the sender's real signature. Every sales routine writes mail through it and never through the mail connector.

## Why it exists

Three problems showed up in production:

1. **Mail connectors can rewrite links.** One widely used connector turned every link in every draft it created into a provider redirect (`google.com/url?q=…`), which looks like phishing and breaks link previews. Only a message whose MIME the mailer builds itself keeps the link exactly as written.
2. **Connectors cannot add the sender's real signature**, so drafts looked machine-made.
3. **Least privilege.** A routine that holds a send-capable mail token can send by mistake. With a mailer, routines hold only read and label permissions. The mailer holds the send permission behind its own switch, and the owner controls that switch.

If the person's mail connector passes a test (a draft with a link and a signature arrives exactly as written), the mailer is optional. Record the test result in the brief.

## Contract

`POST {{MAILER_URL}}` with `Content-Type: application/json`. The routine's environment adds the auth header (`Authorization: Bearer <secret>`). The secret never appears in a prompt.

Request:

| Field          | Type             | Rules                                                                                                       |
| -------------- | ---------------- | ----------------------------------------------------------------------------------------------------------- |
| `action`       | string           | `draft` or `send`. `send` is refused unless the mailer's sending switch is on.                              |
| `to`, `cc`     | array of strings | Plain addresses. At most 5 in total. Each is validated.                                                     |
| `subject`      | string           | Required. For replies, `Re: ` once.                                                                         |
| `text`         | string           | Required. Plain text ending on the sign-off line. Full `https://` URLs. No HTML, no Markdown, no signature. |
| `reply`        | object, optional | `thread_id` and `in_reply_to` (an RFC 822 Message-ID). Puts the message in the existing thread.             |
| `footer_lines` | array, optional  | Lines added under the signature (the mailing address for outreach).                                         |
| `request_id`   | string           | Required. The Queue ID or Owed Replies record id. The idempotency key.                                      |

Response:

```json
{ "ok": true, "draft_id": "…", "message_id": "…", "thread_id": "…", "rfc822_message_id": "…", "duplicate": false }
```

or `{ "ok": false, "error": "<reason>" }`, with reasons including `sending is switched off`, `invalid recipient`, `too many recipients`, `missing field: <name>`, and `suppressed recipient`.

## Steps

1. **Authenticate** the request against the stored secret. Reject anything else with 401 and log nothing from the body.
2. **Validate**: required fields, address syntax, at most 5 recipients, `action` value, and `text` containing no HTML tags.
3. **Idempotency**: look up `request_id` in the mailer's own store (an n8n data table, a key-value store, or an Airtable table). If found, return the stored response with `"duplicate": true` and do nothing else. A retried request never makes a second draft or a second send.
4. **Defence in depth (recommended)**: check every recipient against the CRM's Do Not Contact table (email and `@domain`). Refuse with `suppressed recipient`.
5. **Build the message:**
   - `text/plain` part: `text`, then a blank line, the plain-text signature, and `footer_lines`.
   - `text/html` part: `text` HTML-escaped, line breaks as `<br>`, every `https://` URL wrapped in a plain `<a href>` with the URL as its visible text (no tracking, no redirect), then the sender's **real** signature HTML fetched from the mail provider's send-as settings, then `footer_lines` in small grey text.
   - Headers: `From` = the sender's send-as address and display name; `To`, `Cc`, `Subject`; for replies, `In-Reply-To` and `References` = `<in_reply_to>`, and the provider's thread id.
6. **Act:** `draft` creates a draft in the sender's mailbox. `send` sends only if the sending switch is on; otherwise it returns `sending is switched off` and creates nothing.
7. **Read back** the created message's headers to get the RFC 822 `Message-Id` (strip the angle brackets). Routines store this value; the ingest dedupes on it.
8. **Store** `request_id` with the response, then return it.
9. **Log** a timestamp, the `request_id`, the action, the result, and the error, but never the body or the recipients.

## The sending switch

A single boolean the owner controls, stored where routines cannot write it (an n8n variable or environment setting, not the CRM). Default **off**. Turning it on is a `MANUAL.md` step, done on the day the owner moves Mode to Auto-send.

## Hosting and auth

- **n8n** (default): a Webhook node with header auth, a Code node for validation and MIME, the mail provider's API through an HTTP Request node using the sender's OAuth credential or a service account with domain-wide delegation scoped to compose and send for that one mailbox, and a data table for idempotency.
- The webhook must be reachable from the agent runtime. A self-hosted n8n behind a private network needs a public tunnel for that one path. Add the mailer's host to the routine environment's network allowlist.
- Put the auth secret in the agent runtime's environment so it is added to requests automatically, and in the workflow's credential store. Rotate it if it ever appears in a log.

## Smoke test (a manual-only routine)

Create a routine named `SP · Mailer smoke test`, schedule it for a date far in the future so it never fires on its own, run it by hand after any environment change, and keep it disabled otherwise. It needs no connector, but give it an explicit list: on some platforms an empty list attaches every connector on the account. One harmless read-only connector is enough, with every write tool denied. Use a cheaper model; it only runs a request. Its prompt:

```
Smoke test for the sales mailer. Do exactly these steps and nothing else. Never print request headers or any secret. Send no message and no push notification anywhere.

1. With python, write this JSON to /tmp/mail.json using json.dump:
   {"action": "draft", "to": ["{{OWNER_EMAIL}}"], "subject": "Mailer smoke test (delete me)",
    "text": "Hi,\n\nThis draft came from the sales routine environment through the mailer. Booking link: {{BOOKING_LINK_EXAMPLE}}\n\nCheers,",
    "request_id": "smoke-test-<current UTC timestamp>"}
2. Run: curl -sS -m 60 -o /tmp/resp.json -w '%{http_code}' -X POST {{MAILER_URL}} -H 'Content-Type: application/json' --data @/tmp/mail.json
   The environment adds the Authorization header itself.
3. Print the HTTP status code and the contents of /tmp/resp.json.
4. Reply with two lines: "mailer reachable with environment credential: yes/no" and the draft_id and thread_id (or the error).
```

Pass: the owner finds one draft to themselves, the link is clickable and unchanged, and their real signature sits under "Cheers,". The owner deletes the draft.

Keep the timestamp in `request_id`. A fixed id makes every later run return the first run's response as a duplicate and create nothing, so the test reports "reachable" while proving nothing.
