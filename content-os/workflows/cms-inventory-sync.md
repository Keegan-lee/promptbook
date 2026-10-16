# CP · CMS Inventory Sync

| Setting | Value                                                                                        |
| ------- | -------------------------------------------------------------------------------------------- |
| Trigger | Nightly at `{{INVENTORY_SYNC_TIME \| 02:00}}` in `{{TIME_ZONE}}`                             |
| Reads   | {{CMS \| Sanity}}, **read-only**; Content                                                    |
| Writes  | Content: CMS fields on existing rows; new rows for newly live documents; Status forward only |
| Modes   | Paused, Live                                                                                 |
| Module  | CMS                                                                                          |

## Purpose

Keep one Content row per public CMS document, so the base is the single record of everything published, and so a newly live post can be promoted on social. The CMS stays the source of truth for the document. This job never writes to the CMS.

## Rules

- **Read-only against the CMS.** Use a read token (viewer role). It cannot create, patch, or publish, so a bug here cannot change the site.
- **Natural key:** CMS Document ID = the published id, without any draft prefix. Upsert on it. A document already linked by the blog kit or the article routine is updated, never duplicated.
- **Status moves forward only.** The job may move Approved or Scheduled to Published. It never moves a row backward and never touches Archived or Rejected rows.
- Title and Type are written only when the job creates a row. After that, people own them.
- Never delete a Content row when a document disappears. Add one Review Notes line and leave the row.

## Steps

1. Read the Settings row. Paused: stop.
2. Query the CMS for documents of the types in `{{CMS_DOC_TYPES}}`, with: id, type, title, slug, published state, the live flag `{{CMS_LIVE_FLAG | live}}`, first published time, last updated time. Page through all of them.
3. Build each document's public URL from `{{CMS_URL_PATTERN | <origin>/blog/<slug>}}`.
4. For each document that is **published and live** (a document with no live field counts as live, as the blog kit's gate does):
   - **Existing row** (CMS Document ID matches): update CMS Type, CMS Live, CMS Published At, CMS Updated At, Canonical URL when empty. If Status is Approved or Scheduled, set it to Published.
   - **No row**: create one. Title, Type (mapped from the CMS type), CMS fields, Canonical URL. Status = Briefed when `{{PROMOTE_NEW_POSTS | yes}}` is yes, so the Copy Writer drafts link posts for it; otherwise Published. Brief = `Promote this live post: <excerpt>`. Review Notes: `Created by CP · CMS Inventory Sync on <date> from <CMS id>`.
5. For each document that exists but is **not public** (a draft, or live false): update CMS Live and CMS Updated At on an existing row only. Never create a row for it.
6. For each Content row with a CMS Document ID that the query did not return: add one Review Notes line `<date> · CP · CMS Inventory Sync: document not found in the CMS` (once).
7. Write the Last Run Summary: documents read, rows created, rows updated, rows moved to Published, missing documents.

Batch: write in groups of 10 records. No other limit: the job is read-heavy and runs at night.

## n8n notes

- Sanity: an HTTP Request to the GROQ query endpoint with the read token, or the Sanity community node. Query `*[_type in $types]{_id, _type, title, "slug": slug.current, live, _createdAt, _updatedAt, publishedAt}`. Ids starting `drafts.` are drafts: strip the prefix to find the published id, and treat a published id with no matching published document as not public.
- Airtable upsert: the Airtable node's "Create or Update" with CMS Document ID as the column to match on.

## Test

Publish a `[TEST]` post in the test dataset with the live flag on, point a manual run at the test dataset, and confirm one Content row appears with the right Canonical URL. Run it again: no second row. Unpublish and delete the test post in the CMS (a person does this), then mark the Content row Archived.
