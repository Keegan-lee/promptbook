# CP · One-off: Media Public Links

| Setting | Value                                                                                            |
| ------- | ------------------------------------------------------------------------------------------------ |
| Trigger | Manual, once per batch of exported files                                                         |
| Reads   | `{{FILE_HOST \| Dropbox}}` files under `{{MEDIA_ROOT}}`; Assets                                  |
| Writes  | Assets → Public Media URL, Link Checked At, File Path, Notes; page Assets for multi-page designs |
| Modes   | Manual runs only. No schedule.                                                                   |

## Purpose

The distributor fetches media by URL, so every Asset needs a **direct download** link: a URL that returns the file's bytes, not a preview page. This job makes those links for a batch of files and proves each one works.

## Rules

- Make links only for files in the batch list. Never share a folder, and never touch files outside `{{MEDIA_ROOT}}`.
- Never use a link the job did not just verify.
- Never store an expiring link (an Airtable attachment URL, an S3 presigned URL, a temporary download link).
- Re-exports get a new file with a new version suffix next to the old one. The Asset's Notes keep the old URL for rollback.

## Steps

1. Edit the batch list (n8n: the Files node) with the file paths, and the Asset each one belongs to (Asset Name or External ID). For a multi-page design (a carousel exported page by page), list each page file with the design's Asset and its page number.
2. For each file:
   1. Get the file's existing shared link, or create one. This needs a credential with the sharing scope.
   2. Convert it to a direct download:
      - Dropbox: replace `dl=0` with `dl=1` (or add `raw=1`). The download comes from `dl.dropboxusercontent.com`.
      - Google Drive: `https://drive.google.com/uc?export=download&id=<file id>`. Large files return a warning page instead; host large video elsewhere.
      - S3 or a CDN: the public object URL.
   3. **Verify** with a ranged GET, `Range: bytes=0-1023`. Pass: status 206 or 200 and a `video/…`, `image/…`, `audio/…`, or `application/pdf` content type. A `HEAD` request is not enough: Dropbox answers `application/json` to it.
   4. Pass: write Public Media URL, Link Checked At, and File Path on the Asset (match on External ID or Asset Name). If the Asset already had a URL, add a Notes line with the old one first. A page file creates one page Asset instead: Asset Name `<design asset name> · p<page>`, Kind Carousel Slide, Parent Asset, Page, Status Approved, the link, the same Content and Platform Posts links. Its approval is the parent design's human stamp; the job writes no stamp. Only for a parent Asset that a person has approved.
   5. Fail: write nothing to the Asset. List the file in the run's output with the status and content type.
3. Report: links written, files failed, Asset ids.

## Design exports

When the creative module runs without a file-host write path, a person exports each **approved** design from the design tool into `{{MEDIA_ROOT}}/social/<content id>/`, using the naming convention below, and runs this job on the batch. Never export a design that changed after its approval: approve it again first.

## Naming convention

Keep media paths predictable so a person can find a file from its Asset:

- long-form clips next to their masters: `{{MEDIA_ROOT}}/episodes/<episode id>/clips/<clip id>_v<N>_<YYYY-MM-DD>.mp4`
- promo videos: `{{MEDIA_ROOT}}/promos/<slug>/<slug>_<seconds>s_<16x9 or 9x16>_v<N>.mp4`
- images and carousels: `{{MEDIA_ROOT}}/social/<content id>/<content id>_<kind>_v<N>.<ext>`

## n8n notes

- n8n's built-in Dropbox credential cannot create shared links. Create a **generic OAuth2** credential against Dropbox with the `sharing.write` and `files.metadata.read` scopes, and use HTTP Request nodes against the Dropbox API.
- An agent routine that later fetches these files needs the direct-download domain on its environment's allowlist.

## Test

Run it on one test image and one test video. Paste each URL in a private browser window: the file downloads or plays directly. Attach the image's Asset to a `[TEST]` post and let Distribute's Dry Run name it. Delete the test Asset and say so.
