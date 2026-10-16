# Blog ticket template

Every blog ticket carries this body in its `Brief` field (dedicated table) or its description (existing task table). The Gap Scan and the Refresh Review write it. A person may write one by hand. The Post Writer follows it.

Two kinds of blank:

- `{{…}}` is filled once, at build, from the brief. It is already filled inside the routine prompts.
- `<…>` is filled per ticket by whoever writes the ticket. A ticket with a `<…>` left in it is not Ready.

Keep every heading, in this order, even when a section says `n/a`. The writer and later scans look for them.

## Template

```markdown
## TASK

Create a {{CMS}} draft for this <Kind> ticket. Do not publish. `{{LIVE_FIELD | live}}` stays false.
A person reviews the draft, publishes it, and switches it live.
<Refresh or Correction only: write a draft of the existing document <Existing Doc Id> (<Existing URL>).
Keep its id, type, slug, author, and publish date. Change only what this ticket names.>

## POST FIELDS

- **Title:** <working title; contains the primary keyword naturally; about 50–60 characters>
- **Slug:** <kebab-case, at most {{SLUG_MAX | 64}} characters; new posts only, else "unchanged">
- **Archetype:** <archetype from the brief>
- **Category:** <category slug that exists in the CMS>
- **Author:** {{AUTHOR_DEFAULT}}
- **Reviewer:** {{REVIEWER_DEFAULT}}
- **Target audience:** <audience from the brief>
- **CTA:** <catalog id, or none>
- **Summary:** <one or two sentences: the reader's question, and the shape of the answer>
- **Primary keyword:** <lowercase trimmed keyword>
- **Secondary keywords:** <keyword>; <keyword>; <keyword>
- **Target place:** <place name, or n/a>
- **Refresh scope:** <Confirm, Update, or Expand; or n/a>

## KEYWORD DATA

| Keyword     | Volume              | Difficulty | CPC        | Location id | Tool   | Pulled       |
| ----------- | ------------------- | ---------- | ---------- | ----------- | ------ | ------------ |
| <primary>   | <n or range or n/a> | <n or n/a> | <n or n/a> | <id>        | <tool> | <YYYY-MM-DD> |
| <secondary> | …                   | …          | …          | …           | …      | …            |

SERP (<tool>, loc <id>, <YYYY-MM-DD>): <who ranks, by domain>; winning content type: <guide, list, tool, directory, local results…>;
small site in the top 10: <yes or no>; downgrade: <none, or the reason>.

## OUTLINE

1. Title (rendered as the H1).
2. Answer box: 40–60 words answering "<the question in the reader's words>". Conclusion first. Verdict or number in bold.
   <Omit only for archetypes the brief marks "no answer box".>
3. Disclaimer: <yes (legal, financial, health) or no>.
4. Intro: 2–3 sentences. Who this is for and what they will be able to do.
5. H2: <question a reader asks>
   - <what the section must establish; facts to use, with their source numbers from SOURCE NOTES>
6. H2: <question>
   - <…>
7. Mid-article CTA: <catalog id>. After the reader has got something, not before the first H2.
8. H2: <question> <add or remove sections to fit the archetype's recipe>
9. FAQ: 3–6 questions. Plain answers under 80 words. None restates the answer box.
10. Sources and corrections: last. Every source in SOURCE NOTES that the post uses.

Recipe for <archetype>: <the block order after the intro, copied from the brief's recipe table>.

## INTERNAL LINKS

At least three, in body text, each a real URL that resolves today:

- Hub: {{SITE_ORIGIN}}{{BLOG_PATH | /blog}}/<category slug>
- Sibling post: <live post URL on a related question>
- Glossary term or landing page: <URL, or "none exists yet: suggest one in Agent Notes">
- Product route: <path from the CTA catalog>

## CTA RULE

Use catalog id <id> for the end-of-post CTA and the mid-article band. Label and URL come from the Site brief's catalog, never from this ticket.
Never name a product, offer, or page the catalog does not list. <If the archetype takes no CTA: "none; no CTA block".>

## SEO RULES

- The primary keyword appears in the title, the meta title, the answer box's first sentence, and the first 100 words. Once each is enough. No stuffing.
- Meta title at most {{META_TITLE_MAX | 60}} characters, counting the suffix the site appends ({{TITLE_SUFFIX | none}}). Meta description 120–160 characters.
- One H1 (the title). The body starts at H2.
- Secondary keywords go into H2s or body text only where they read naturally.
- The slug is never a reserved segment ({{RESERVED_SEGMENTS}}) and never equals a category slug.

## AEO RULES

- The answer box sits directly under the title and can be quoted alone.
- H2s are questions readers actually ask: from autocomplete data, the SERP, or the brief's audience language.
- FAQ answers are plain sentences, not "click here".
- No statistic ships without a source that was opened. No source, no claim: cut it.
- {{#if AUTHOR_IS_AI}}The author is an AI persona: the `reviewer` field names {{REVIEWER_DEFAULT}}. No reviewer, no draft.{{/if}}

## GEO RULES

- Use the typed blocks (answer box, FAQ, how-to steps, comparison table, sources) so the page emits its structured data. Never fake them with plain headings when the block exists.
- Name organisations, programmes, rules, and places by their official names, and link their official pages.
- Never type a number from this list: {{NEVER_TYPE_NUMBERS}}. Describe the landscape in words that stay true. Name entities freely.
- <Geo only: at least {{GEO_MIN_FACTS | 3}} locally specific facts (a named authority, a local rule, a local programme, a named local organisation), each with its source below. Never produce this page by find-and-replace from another place's page.>

## IMAGES

- Cover: {{IMAGE_STYLE}}, per the brand pack at {{BRAND_PACK_LOCATION}}. Size {{COVER_SIZE | 1200x630}}. Text on the image: {{TEXT_ON_IMAGES | no}}.
- Alt text: one plain sentence that says what the image shows. No keyword lists.
- Brief for the image: <one or two sentences: the scene, the subject, what to leave out>.
- If no image can be made, leave the cover empty and say so in Agent Notes.

## SOURCE NOTES

Number every source. The writer opens each one before using it.

1. <claim it supports> — <URL> — <publisher> — <found by: routine name, YYYY-MM-DD>
2. …
   Evidence, not copy (never typed into the post):

- <Geo only: supply tier, the query, read 1 = <n>, read 2 = <n>, <YYYY-MM-DD>>
- <Refresh only: what the review found, with the URLs checked and the dates>

## ACCEPTANCE

- [ ] A {{CMS}} draft exists, `{{LIVE_FIELD | live}}` is false, and no published document was created or changed.
- [ ] The answer box is 40–60 words with the conclusion first (or the archetype has none).
- [ ] H2s are questions. The FAQ has 3–6 entries.
- [ ] At least three internal links, all resolving.
- [ ] The CTA is the catalog entry named above, or none.
- [ ] Every statistic has a source in the sources block, and every source was opened.
- [ ] No number from the never-type list appears anywhere, including captions and the FAQ.
- [ ] Meta title, meta description, and slug meet the SEO rules. The slug did not collide.
- [ ] Every reference (category, author, reviewer, glossary term, featured record) resolves to a published document.
- [ ] A cover with alt text is attached, or Agent Notes says it is missing.
- [ ] The ticket shows the Draft URL, the Preview Path, and Status Draft Ready.
```

## What each kind adds

| Kind       | Must also have                                                                                                                                                                                                                                                                            |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Geo        | Target place, its location id, the supply tier that qualified it, and at least `{{GEO_MIN_FACTS \| 3}}` sourced local facts in SOURCE NOTES before the ticket is created. Counts from your own data are evidence only.                                                                    |
| Depth      | The cluster, and the long-tail questions the post answers as H2s (from autocomplete or related keywords, with their labels).                                                                                                                                                              |
| Refresh    | Existing Doc Id, Existing URL, Refresh Scope, and a list of what to change and why. The draft keeps the id, type, slug, author, and publish date, and sets the review date to the day it was written. A fact that was wrong also gets a dated entry in the sources and corrections block. |
| News       | The date of the event, the primary source (the release, the notice, the announcement), and the durable query a reader will search later (the programme or rule name, not the headline). Once live, news is corrected, never rewritten.                                                    |
| Correction | The exact sentence that is wrong, the correct statement with its source, and the correction entry text (`<YYYY-MM-DD>: <what changed and why>`). The draft changes nothing else.                                                                                                          |

## Rules for whoever writes a ticket

- Every number in KEYWORD DATA is copied from a tool response or a ledger row, with its tool, location id, and date. Never typed from memory, never rounded, never estimated.
- Every URL in SOURCE NOTES and INTERNAL LINKS is one you opened or read from the CMS this run.
- Never invent a place, a person, an organisation, a programme, a quote, a slug, or a record id.
- The `Primary keyword:` line matches the `Primary Keyword` field exactly (dedicated table). In the existing-task-table model, the line **is** the dedupe key.

## The `Primary keyword` line (existing task table)

When tickets live in a description field, the line must read exactly:

```
- **Primary keyword:** <lowercase trimmed keyword>
```

- One line, under `## POST FIELDS`, starting at the first character of the line.
- The keyword is lowercase, trimmed, with no quotes and no trailing punctuation.
- Never reformat it, bold it differently, or move it. Every later run parses this exact shape. A ticket whose line does not parse is reported as `DEDUPE-UNVERIFIED`.
