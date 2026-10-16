# Publication spec

A CMS-backed publication whose job is to turn search and answer-engine traffic into the business’s next step. Posts keep their URLs. Category hubs absorb new topics. Locales layer on without changing slugs.

This file is the build contract. Complete **Interview** and write **Site brief** before creating schema, routes, or copy. If an answer is missing, ask. Do not invent the business, its offers, or its calls to action.

## Interview

Ask these in order. Record the answers in the Site brief. Skip a later section only when an earlier answer makes it impossible (no second language, no paid research, no live operational data).

### Business

1. What is the business, in one sentence a stranger could repeat?
2. Who is it for? Name each audience (the person who buys, the person who supplies, the person who partners).
3. What should a reader do after a useful article? Rank the outcomes. Typical outcomes are: start a trial, book, apply, request a quote, join a list, open a tool.
4. What must the publication never claim? (Live counts that drift, prices that change, legal or medical outcomes, competitor facts you have not sourced.)
5. What is the canonical site origin (`https://…`)? Which paths are private and must stay out of the index (`/dashboard`, `/api`, and so on)?
6. Who is the publisher (legal name, logo, sameAs profiles)? Who may review AI-assisted articles?

### Calls to action

Build a **CTA catalog** from the answers. Every entry needs an id, a button label, a path or URL, the audience, and the kind of article that should use it.

7. What are the primary conversions, and the URL for each?
8. For each conversion, what is the one-line reason a reader who just finished an article would care?
9. Which conversions are secondary (a text link under the button, not the button)?
10. Is there a newsletter? What is the default heading and body, and does any topic need its own?
11. Should any article have no call to action?

Do not ship a hardcoded fallback that names a product the interview did not list. The fallback is the catalog entry marked default.

### Rich content

Rich content is a block that reads **live operational data** at request time, so the article is also a path into the product. Ask what the business can embed. Do not assume a map.

12. What live objects can an article point at? Examples: a bookable listing, a product, a public dataset, a calculator, an embeddable map, a store locator. For each, name the system of record and the stable public id (usually a slug).
13. For each object, what must the card show (name, price, availability, location) and what must it never imply (that an unavailable record is open)?
14. Does the embed need its own landing page (a city, a catalog, a dataset)? If yes, that page is the attribution target: the embed’s “open” and “copy link” actions point there, and that page is the backlink.
15. Which numbers on that landing page are computed on each request? Those numbers are never typed into article copy, captions, FAQs, or stat blocks. Describe the landscape in words. Name entities freely.
16. Are there interactive tools that are not live queries (a cost estimator with fixed, per-unit, and range lines; a checklist the reader downloads)?

If the answer to 12 is “nothing,” the publication still ships the static blocks (answer, FAQ, table, steps, quote, sources). It does not invent an embed.

### Categories and sections

The index is a set of **rails**, not a reverse-chronological feed. Each rail is one kind of content, so a reader or a crawler sees the whole offer in one hop. The archive sits under the rails and lists every live post.

17. What are the sections a first-time visitor should see, in order? For each: title, one-sentence description, and whether it is a named rail or an ordinary hub.
18. Which section is the business’s own story? Which are customer stories, news, explainers, locations, research?
19. Only one category may feed each named rail. Which categories are ordinary hubs (nav, hub page, archive filter) with no rail of their own?
20. What is the pillar of each hub — the sentence every post in it has in common?
21. Tag pages exist, but they stay `noindex` until five live posts. Confirm that threshold or set another. Do not index a tag at one post.

### Trust, language, and research

22. Who are the authors? For each: name, role, credentials, and public profile links.
23. Will any author be an AI persona? If yes, every such post needs a human `reviewer`.
24. Does the business want a glossary of terms it should be cited for? If yes, each term needs a short definition of at most 300 characters.
25. Which languages ship on day one? Translations are fields on the same document, not new URLs, until locale routing is turned on.
26. Is any article paid or metered? If yes, the free portion and the paid portion are split on the server. The paywall is declared in structured data. Gated HTML is never sent to the browser.

### Keyword research

27. Which keyword tool is connected? Default: Ubersuggest. If it is not connected, connect it before any category or post is drafted.
28. What market and language should queries use (country, language)?
29. Who owns the keyword list (a CMS field, a sheet, a keyword-list inside the tool)?

## Site brief

Fill this after the interview. The build reads only this section plus the requirements below.

Store the filled brief where the writing kit can read it (`KICKOFF.md` in this folder). Put a `Version X.Y · YYYY-MM-DD` line at its top and make `END OF SITE BRIEF` its last line: the writing routines read it on every run and use the marker to detect a truncated read.

|                                  |                                        |
| -------------------------------- | -------------------------------------- |
| Business, one sentence           |                                        |
| Audiences                        |                                        |
| Origin                           |                                        |
| Private path prefixes            |                                        |
| Publisher                        |                                        |
| Human reviewers                  |                                        |
| Default CTA id                   |                                        |
| Newsletter                       | yes / no                               |
| Rich embeds                      | none, or a list of object types        |
| Landing pages for embeds         | none, or a list                        |
| Numbers that must never be typed |                                        |
| Named rails (one category each)  |                                        |
| Ordinary hubs                    |                                        |
| Tag index threshold              | 5, unless changed                      |
| Languages now                    |                                        |
| Paid content                     | off, unless the interview turned it on |
| Keyword tool and market          |                                        |

### CTA catalog

| id  | Label | URL | Audience | Use when | Default |
| --- | ----- | --- | -------- | -------- | ------- |
|     |       |     |          |          |         |

### Category plan

| Slug | Title | Rail (`standard` or a rail id) | Description | Pillar | Primary keyword |
| ---- | ----- | ------------------------------ | ----------- | ------ | --------------- |
|      |       |                                |             |        |                 |

### Embed plan

| Block | System of record | Public id | Card shows | Landing page |
| ----- | ---------------- | --------- | ---------- | ------------ |
|       |                  |           |            |              |

## Requirements

### Schema

Documents live in the CMS. The app renders them. A document id must not contain a dot (apart from the CMS's own draft prefix): some CMSs treat a dotted id as a private path, and anonymous readers then get nothing. Use `{type}-{slug}`.

Public queries return a post, glossary term, or location page only when it is **published and `live`**. `live` defaults to false. A legacy document with no `live` field counts as live. Anything else is a 404, not a redirect and not a soft gate. The same filter feeds the index, hubs, author pages, sitemap, feeds, related posts, and every post count the site shows (section nav, hubs, author pages).

- **A real 404.** The gated URL must answer with HTTP status 404, not a 200 page that says "not found". A streamed loading state in front of the page can turn a not-found into a 200 (a soft 404). Check the status code, not the page.
- **Public datasets.** If the CMS dataset is publicly readable, a published document can be read through the CMS's API even while the site 404s it. The live flag hides it from the site, not from the API. Never publish anything that must stay unannounced, live flag or not. Drafts stay private.
- **Two gates, two clicks.** Publishing and switching `live` on are separate, both by a person. Every message that hands over a draft says so, because a forgotten switch is a silent 404.
- **References.** A draft may reference only published documents. A reference from a draft to a draft breaks when the post is published. A supporting document a draft needs (a glossary term, a location page) is created published with `live: false`, so the reference holds and the live gate keeps it hidden until a person switches it on.

| Document                 | Required fields                                                                                                 | Notes                                                                                                                                                                                                              |
| ------------------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Post                     | title, slug, excerpt, body, category (ref), author (ref), `live`, `publishedAt`, `lastReviewed`, `ctaType`, seo | `ctaType` is an id from the CTA catalog, or `none`. `reviewer` (ref author) is required when the author is an AI persona. Optional: cover image + alt, cover caption, tags, language, `translations[]`, paywall.   |
| Category                 | title, slug, description (required, ≤400 characters), `kind`, order, seo                                        | `kind` is `standard` or exactly one named rail from the brief. Also: pillar heading, pillar body, hub stats (durable strings, not live counts), `sidebarCta`, newsletter heading and body, language, translations. |
| Author                   | name, slug, role, bio                                                                                           | credentials[], links (site, and whichever profiles the interview named), `isAi`, language, translations.                                                                                                           |
| Glossary term            | term, slug, `shortDefinition` (≤300), definition, `live`                                                        | Optional related terms. Omit the whole type if the interview declined a glossary.                                                                                                                                  |
| Location or dataset page | title, slug, place or subject fields, intro, answer, body, faq, `live`, `lastReviewed`                          | Only if the embed plan has a landing page. Holds the canonical URL the embed attributes. Optional: nearby pages, search keys. No stored copy of a count the page recomputes.                                       |
| Research                 | title, slug, summary, `kind`, `publishedAt`, paywall, data tables                                               | Only if paid or primary-source research is in the brief. Each table has its own price and a file.                                                                                                                  |

**Sidebar CTA object** (on the category, overridable nowhere else): eyebrow, heading, body, `ctaLabel`, `ctaUrl`, optional secondary label and URL, tone. Render it only when heading, label, and URL are all non-empty. Otherwise use the catalog entry for the post’s `ctaType`. If that is `none` or missing, use the default catalog entry.

**SEO object** on every public document: `metaTitle` (≤70, aim 50–60), `metaDescription` (aim 120–160), keywords[], optional Open Graph image 1200×630 with alt. Stega and draft markers never appear in metadata.

**Body blocks.** Every post template starts unpublished (`live: false`) and lays blocks out in this order: answer box, disclaimer when the topic is legal or financial, intro, sections, a conversion block mid-article, FAQ, sources last.

| Block                   | When                                                                                                                                                               | Structured data                                 |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| Answer box              | Every post. 40–60 words. Verdict or number in bold.                                                                                                                | Joins `FAQPage`. Heading id `the-short-answer`. |
| Disclaimer              | Legal or financial topics. Reads `lastReviewed` and `reviewer`.                                                                                                    |                                                 |
| Stat callout            | 2–4 figures. Each has a note and a `sourceUrl` someone opened.                                                                                                     |                                                 |
| FAQ                     | End of post. Per-block opt-out of schema.                                                                                                                          | `FAQPage`                                       |
| How-to steps            | Procedures.                                                                                                                                                        | `HowTo`                                         |
| Comparison table        | Real `<table>`.                                                                                                                                                    |                                                 |
| Inline CTA              | Mid-article. Label and URL required, or the block does not render.                                                                                                 |                                                 |
| Key term                | Points at a glossary term.                                                                                                                                         |                                                 |
| Sources and corrections | Mandatory when the post cites a regulation, price, programme, or statistic. Corrections update `dateModified`.                                                     |                                                 |
| Pull quote              | Attributed.                                                                                                                                                        |                                                 |
| Interview Q&A           | Real speaker labels.                                                                                                                                               |                                                 |
| Featured record         | Only if the embed plan has a record type. Referenced by public id. Batch-load every id on the page once. Never fetch inside the block.                             |                                                 |
| Live embed              | Only if the embed plan has one. Iframe or component plus open, copy-link, and “make your own” when the product supports it. Attribution links to the landing page. |                                                 |
| Business spotlight      | A real organisation, with a followed outbound link.                                                                                                                | `LocalBusiness`                                 |
| Media                   | Player, chapters, transcript shipped with the page.                                                                                                                | `VideoObject` or `PodcastEpisode`               |
| Checklist download      | Email for a file. Consent before any analytics.                                                                                                                    |                                                 |
| Cost estimator          | Fixed, per-unit, and range lines. No formulas stored as prose. Ends in a catalog CTA.                                                                              |                                                 |
| Paid gate               | Marker. Everything after it is withheld.                                                                                                                           | Paywall declaration on the article              |

Blocks that need CMS references (glossary, research, authors, files) are dereferenced in the page query. Blocks that need operational data receive it from page context, one batch per body. A live embed stores only its share token or public id; its filters live in the product. The CMS stores labels, never the product's internal slugs, so a change to the product's taxonomy never needs a content migration.

**Translations.** One document per topic. `translations[]` holds per-language title, excerpt, body, and SEO. The default language stays on the top-level fields. Resolve by overlaying the requested locale and falling back per field. Do not advertise hreflang until the locale route exists.

**Preview.** `?preview=<secret>` rewrites to a noindex preview route. Compare the secret in constant time. Preview sets `noindex, nofollow` and `X-Robots-Tag`, and is absent from the sitemap. It may read drafts when the server holds a read token; without one it shows the published-but-not-live document. A banner shows both gate flags. The secret lives in the site's environment, never in a ticket, a prompt, or a chat message.

**Bulk edits.** Any script or routine that changes many documents at once (a backfill, an enrichment pass) writes to drafts only, runs as a dry run unless told to apply, takes the dataset as an explicit argument, and enforces the same guards as the writer (no dotted ids, no live counts, sources for every number, a human reviewer on AI bylines). Test the guards like code.

### Categories and sections

- `/blog` renders one rail per named `kind`, in `order`, then an archive of every live post. A rail with no live posts is omitted. A switch in admin may hide a rail without deleting the category.
- `/blog/{category-slug}` is the canonical page for that topic. It is indexed whenever it has a description and at least one live post. It carries the pillar, the hub stats, and the category CTA.
- A post slug that equals a category slug is shadowed by the hub. The CMS warns. The backfill reports it.
- Reserved first segments under `/blog`: `authors`, `tag`, `preview`, `page`, plus any landing-page prefix from the embed plan (`map`, `locations`, or whatever the brief named). Do not use those as category or post slugs.
- Tags: `/blog/tag/{tag}` is `noindex` until the threshold (default five live posts). Below the threshold, omit it from the sitemap.
- Authors: `/blog/authors/{slug}` lists that person’s live posts and emits `ProfilePage` + `Person`.
- Glossary, when in the brief: `/glossary` and `/glossary/{slug}`.
- Embed landing pages, when in the brief: their own path, linked from the embed, not stuffed into a post URL.

### Configurable calls to action

- The catalog from the brief is the only set of destinations. Adding a destination is data (a catalog entry), then a CMS option, not a new component.
- Three surfaces read it:
  1. **Sidebar** on a post: category `sidebarCta` if complete, else the post’s `ctaType`, else the default.
  2. **End of post:** the post’s `ctaType`.
  3. **Inline band and estimator:** an explicit label and URL. Prefer a catalog URL. Reject empty labels.
- Internal URLs may be site paths (`/pricing`). External URLs must be `https`. Validate both.
- Every rendered CTA and embed root exposes `data-track-block` and `data-track-id`. One delegated listener emits click and view events only after consent. Attribute later conversions to the first-touch post and the last-touch block. Do not send personal data in the event.

### SEO

- Unique title and description per URL. Title about 50–60 characters. Description about 120–160. One H1, matching the title the reader sees.
- Canonical URL on every indexable page. Paginated archives are self-canonical (`/blog?page=2`) and use real links.
- Open Graph and Twitter cards. Cover or `ogImage` is a real image with alt text. Hero images use priority and a correct `sizes`.
- `lastReviewed` is the visible “Reviewed” date and the input to `dateModified`, ahead of CMS update time, ahead of `publishedAt`.
- Review cadence is part of the brief. Explainers that cite rules or prices are reviewed on a schedule. News is corrected in the sources block, not silently rewritten. Location pages show the date of the underlying data.
- Hubs are the internal-link target for a topic. Posts link to their hub, to glossary terms they define, and to the landing page of any embed they use.
- Old URLs stay. Changing a slug requires a redirect. Do not redirect a gated 404.
- Preview, drafts, thin tag pages, and private prefixes are `noindex`.

### AEO

Answer engines cite a page when the answer is extractable, sourced, and fresh.

- The answer box is above the fold, 40–60 words, with the conclusion in the first sentence.
- H2s are questions the reader actually asks.
- FAQ answers are plain sentences, not “click here.”
- One `FAQPage` graph per page, merging the answer box and every FAQ block that opted in.
- Statistics without a fetched source do not ship.
- Author pages and `reviewedBy` are the experience signal. An AI byline without a human reviewer does not ship.
- Glossary terms use `DefinedTerm` inside a `DefinedTermSet` so the short definition is what gets quoted.

### GEO

Generative engines need a machine-readable map of the site, not only HTML.

- JSON-LD is server-rendered. The page graph is the document graph plus whatever the body contributes.
- Article: `BlogPosting` with headline, dates, author, publisher, `mainEntityOfPage`, `articleSection`, `inLanguage`.
- Hub: `CollectionPage` of the posts it contains.
- Index: `Blog`.
- Author: `ProfilePage` and `Person`.
- Location or dataset landing page: `Dataset` (or `Place` when the brief is a location and there is no downloadable dataset).
- Glossary: `DefinedTerm` / `DefinedTermSet`.
- A declared paywall sets `isAccessibleForFree: false` and `hasPart` with the gated selector. An undeclared paywall is cloaking. Do not declare a paywall while the payment switch is off and the body is fully public.
- `WebSite` JSON-LD points at `/llms.txt`.
- The HTML head links `/llms.txt` as `rel="alternate"` `type="text/plain"`.
- Speakable or answer-first content is the answer box, not a hidden block.

### `llms.txt`

Serve `https://{origin}/llms.txt` as `text/plain` or markdown, cacheable, and outside any auth proxy.

Required shape:

1. H1: the publication or business name from the brief.
2. A blockquote of the one-sentence description.
3. A short prose paragraph: who it is for, what the site does, how to get support.
4. `## AI & Discovery` linking this file, the sitemap, and any sibling properties the brief names.
5. `## Documentation & Guides` for durable explainers (pricing, policies, the blog, the glossary, RSS).
6. One section per audience or product surface from the brief, each bullet a title, absolute URL, and a one-line description.
7. No secrets, no private URLs, no live counts.

Update the file when a public section is added. Put the URL in the sitemap at a low priority.

### `robots.txt`

Serve `/robots.txt` as `text/plain`, cacheable.

- Comment lines naming the AI index (`/llms.txt`) and the sitemap.
- Explicit `Allow: /` for retrieval crawlers: GPTBot, ChatGPT-User, OAI-SearchBot, ClaudeBot, anthropic-ai, PerplexityBot, Google-Extended, Applebot-Extended, cohere-ai, YouBot, Bytespider.
- `User-agent: *` with `Allow: /` and `Disallow` for every private prefix in the brief.
- A final `Sitemap: https://{origin}/sitemap.xml` line.
- Do not disallow `/blog`, `/glossary`, `/llms.txt`, or `/sitemap.xml`.
- Do not put the preview secret, or any preview path pattern that would hide public posts, in `Disallow`. Preview is `noindex` at the page.

### Sitemap

`/sitemap.xml` is generated from the same public filter as the site. Include only:

- `/blog`
- each live post
- each category hub that has a description and at least one live post
- each author with at least one live post
- glossary index and live terms, when the glossary exists
- embed landing pages that are live, when that type exists
- tag URLs at or above the threshold
- `/llms.txt`
- RSS feed URLs (`/feed/blog`, and `/feed/blog/{category}` when a hub has posts)

`lastModified` is `lastReviewed`, else CMS update time, else `publishedAt`. Exclude preview, drafts, `live: false`, private prefixes, and under-threshold tags. When locale routing is on, emit one entry per translated URL with alternates. Until then, emit the default language only.

Also emit RSS for the blog and for each category, using the same filter.

### Keyword research

Connect a keyword tool before drafting. Ubersuggest is the default. The agent may use another tool only when the brief names it and it can return volume, difficulty, and related queries for the brief’s market.

Before a category or a post is written:

1. Run keyword discovery for the pillar sentence and the audience language.
2. Pick one primary keyword and a short supporting set. Record volume and difficulty next to the choice. Do not invent either number.
3. Store the primary keyword on the category or on the post’s SEO keywords, and use it in the H1, the meta title, and the answer box without stuffing.
4. Reject a topic that duplicates a live post’s primary keyword. Point the draft at the existing URL instead.
5. Re-check the primary keyword when `lastReviewed` is updated. If the query has collapsed, say so in the review note. Do not silently retarget the URL.

The tool is for research. It does not publish, and it does not overwrite a live document.

### Measurement and admin

- Admin can hide each index rail, and can turn paid content off. Paid content defaults off. While it is off, hide the paid rail and every gate.
- CTA and embed events wait for analytics consent.
- Server-side logs cover agents that never run the page script (paid-content quotes, when that exists).

### Definition of done

- The Site brief is filled and matches the CMS: catalog, categories, rails, embeds.
- A fixture post in every rail renders the answer box, a mid-article CTA from the catalog, and sources.
- A `live: false` published post answers HTTP 404 (check the status code) and is absent from the sitemap.
- A category CTA with a missing URL falls back to the default catalog entry.
- A live embed, if any, loads operational data in one batch and does not repeat a typed count.
- `/robots.txt`, `/sitemap.xml`, and `/llms.txt` match this spec against the origin in the brief.
- Keyword choices for the launched categories cite the tool, with volume and difficulty, in the brief or the CMS.
