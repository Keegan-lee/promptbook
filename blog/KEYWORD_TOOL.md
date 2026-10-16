# Keyword tool contract

Every blog routine talks to the keyword tool through this contract. It names eight capabilities, maps each one to a real endpoint, and sets the thresholds and budgets that decide whether a keyword earns a ticket.

Ubersuggest is the default. Semrush, Ahrefs, DataForSEO, and Google Keyword Planner are mapped below. Any other tool gets the same table filled in during Phase 4 of the kickoff.

Keyword tools bill per call or per credit (`_shared/ROUTINES.md`, "Metered APIs"). Treat every call as money.

## Rules for every call

1. **Label every number.** A volume, difficulty, CPC, authority score, or ranking position is written with its tool, its location, and the date it was pulled: `vol 320 · diff 22 · CPC 1.10 (Ubersuggest, loc <id>, 2026-03-02)`. A number without a label does not get written anywhere: not in a ticket, the ledger, a brief, or a summary.
2. **Never guess a location id.** Resolve it with the tool (C1) and store it in the brief and on the settings row. Even ids you think you know are read from the tool, not from memory.
3. **Ledger first.** Before a paid call, look the keyword up in the Keyword Ledger (`SCHEMA.md`) on keyword + location id + language. A row pulled within `{{LEDGER_TTL_DAYS | 90}}` days is reused with its original label and date. A row marked `Zero-volume verified` is never queried again.
4. **Allowed endpoints only.** Each routine lists the endpoints it may call. Everything else is forbidden. The expensive endpoints are forbidden by name.
5. **Budget and quota.** Check the remaining quota at the start of the run when the tool exposes it. Count every call against the routine's per-run budget. Stop cleanly when either runs out. A stop for budget is a normal outcome, not an error.
6. **One retry.** A failed call is retried at most once per run. Then the keyword is left for the next run and named in the summary.
7. **Missing is missing.** When a tool does not return a metric, write the field blank and label it `n/a (<tool>)`. Never estimate, model, or borrow a number from another tool to fill the gap.

## The eight capabilities

| #   | Capability               | Input                                    | Returns                                               | Used by                                      |
| --- | ------------------------ | ---------------------------------------- | ----------------------------------------------------- | -------------------------------------------- |
| C1  | Resolve location         | a place name                             | the tool's location id and its parent hierarchy       | kickoff, Gap Scan (each new place)           |
| C2  | Keyword overview         | keyword, location id, language           | volume, difficulty, CPC                               | Gap Scan, Refresh Review                     |
| C3  | Related keywords         | 1–3 seed keywords, location id, language | related keywords with volume and difficulty           | Gap Scan (long tail)                         |
| C4  | Autocomplete             | seed keywords, country, language         | autocomplete phrases, usually without metrics         | Gap Scan (long tail, question H2s)           |
| C5  | SERP analysis            | keyword, location id, language           | the top results: URL, domain, authority, content type | Gap Scan, Refresh Review                     |
| C6  | Domain authority         | the site's bare domain                   | the tool's authority score for the domain             | kickoff, Refresh Review (monthly)            |
| C7  | Domain and page keywords | domain or full page URL, location id     | keywords the domain or page ranks for, with position  | Gap Scan (striking distance), Refresh Review |
| C8  | Quota check              | none                                     | used and remaining allowance                          | every routine, at the start                  |

C3 and C4 are both long-tail sources. C4 is cheap but returns no metrics: every C4 phrase you want to use still needs a C2 call (or a fresh ledger row) before it goes into a ticket.

## Ubersuggest (default)

Parameter names are the Ubersuggest connector's. Location ids are Google location ids.

| #   | Endpoint              | Parameters                                                                 | Notes                                                                                                                                                                                        |
| --- | --------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| C1  | `location_suggest`    | `query` (place name), `limit`, `lang`                                      | Returns matching countries, regions, and cities with their `locId`. Pick the match whose hierarchy is right. If none matches, say so; do not guess.                                          |
| C1  | `location_details`    | `location_ids` (list)                                                      | Confirms the name and parent hierarchy of ids you already hold. Use it at build to double-check the market id.                                                                               |
| C2  | `keyword_overview`    | `keyword`, `locId`, `language`                                             | Volume, CPC, SEO difficulty (SD), paid difficulty. `language` only applies with `locId`. Never omit `locId` for a market query.                                                              |
| C3  | `keyword_suggestions` | `keywords` (1–3 seeds), `locId`, `language`                                | Flat list of related keywords with metrics.                                                                                                                                                  |
| C3  | `match_keywords`      | `keywords`, `locId`, `language`, `limit`, `sortby`, `domain`               | Paged alternative to `keyword_suggestions`. `domain` is optional and must be a real domain, never a seed phrase.                                                                             |
| C4  | `google_suggestions`  | `keywords` (up to 10 seeds), `country` (two-letter, lowercase), `language` | Hundreds of autocomplete phrases per seed. No metrics.                                                                                                                                       |
| C5  | `serp_analysis`       | `keyword`, `locId`, `language`, `limit`                                    | Top results with metrics. Pass `limit: 10`; you judge the top 10.                                                                                                                            |
| C6  | `domain_overview`     | `domain`, `locId`, `language`                                              | Traffic, organic keyword count, domain authority, backlinks summary.                                                                                                                         |
| C7  | `domain_keywords`     | `domain`, `locId`, `language`, `limit`, `searchType: organic`              | Keywords the domain ranks for, with position, volume, difficulty.                                                                                                                            |
| C7  | `page_keywords`       | `page` (full URL), `locId`, `language`, `limit`                            | Keywords one page ranks for.                                                                                                                                                                 |
| C7  | `page_overview`       | `page`, `locId`, `language`                                                | One page's organic keywords and traffic. Use instead of `page_keywords` when you only need the summary.                                                                                      |
| C7  | `domain_top_pages`    | `domain`, `locId`, `language`, `limit`                                     | Top pages by estimated traffic. Optional, for the refresh review.                                                                                                                            |
| C8  | `user_limits`         | none                                                                       | **Free.** `usage.account.reports` is today's searches against the daily limit. `usage.account.monthly_keyword_metrics_updates` is the monthly credit pool the expensive endpoints draw from. |

**Allowed in routines:** `user_limits`, `location_suggest`, `location_details`, `keyword_overview`, `keyword_suggestions`, `match_keywords`, `google_suggestions`, `serp_analysis`, `domain_overview`, `domain_keywords`, `page_keywords`, `page_overview`, `domain_top_pages`.

**Allowed only with the owner's say-so, at build time:** `competitors` (an async report that polls for up to 40 seconds; useful once when choosing clusters), `content_ideas` (top pages by shares for a topic), `keyword_lists`, `create_keyword_list`, `add_keywords_to_list` (only if the brief asks the tool's lists to mirror the ledger; plans cap the number of lists).

**Forbidden, always:**

| Endpoint                                                  | Why                                                                                                |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `generate_article`                                        | Costs 100 credits from the monthly pool. The engine writes its own drafts.                         |
| `keyword_metrics` (any `metric`)                          | `search_difficulty` recalculation draws on the monthly metrics quota. The overview's SD is enough. |
| `article_title_suggestions`, `get_article`                | Part of the article-generation flow. Not needed.                                                   |
| Project, site-audit, backlink, and brand-visibility tools | Out of scope. Some consume project slots or long-running reports.                                  |

Assume every allowed call except `user_limits` counts as one search against the daily limit. The tool does not say which calls are free, so count them all.

## Per-run budget

The budget is a ceiling on paid calls per run. The quota floor is what the engine leaves untouched for people using the tool by hand.

| Routine                 | Budget                          | Typical spend                                                                                                             |
| ----------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Kickoff Phase 4         | `{{BUILD_CALL_BUDGET \| 25}}`   | 1–2 location, 1 domain authority, 1 per excluded term to verify, 3 overviews and 1 SERP for the pilot                     |
| `BL · Content Gap Scan` | `{{SCAN_CALL_BUDGET \| 40}}`    | up to 4 location, about 15 overviews, 3–6 related-keyword calls, 1–2 autocomplete calls, up to 8 SERPs, 1 domain keywords |
| `BL · Refresh Review`   | `{{REFRESH_CALL_BUDGET \| 20}}` | 1 domain authority (monthly), 1 page-keywords call and at most 1 overview per post reviewed                               |
| `BL · Post Writer`      | 0                               | The writer never calls the keyword tool. It uses the numbers on the ticket.                                               |

At the start of each run:

1. Call C8. Read the remaining daily allowance (`limit − used`).
2. Effective budget = the smaller of the routine's budget and `remaining − {{QUOTA_FLOOR | 10}}`.
3. If the effective budget is zero or less, write a one-line run summary (`quota low: <used>/<limit> (<tool>, <time>)`) and stop. Do not write an error row.
4. If the tool has no quota endpoint, count calls yourself against the routine budget.

Spend the budget in this order: confirm candidates you already have (C2 on fresh seeds), then SERP checks on survivors (C5), then long-tail expansion (C3, C4). A run that runs out mid-way writes the tickets that fully qualified and leaves the rest in the ledger as `Candidate`.

## Thresholds

These are rules of thumb. The brief holds the values. The settings row holds the live copy, so the owner can tune them without editing a routine.

### Minimum volume

`{{MIN_VOLUME | 30}}` monthly searches in the brief's market, for the primary keyword.

- 30 suits a single-country market in a major language. Use 10–20 for a small country, a minority language, or a place page measured at city level. Use 50–100 when there are always more qualifying candidates than weekly slots.
- Tune it from the run summaries. More qualifying candidates than slots for three weeks running: raise it by 10. A shortfall for three weeks running where volume was the main rejecting metric: lower it by 10.
- Secondary keywords need volume above 0 and the same difficulty ceiling. They do not need the minimum.

### Maximum difficulty, scaled to domain authority

A young site cannot rank for terms the big sites fight over, however good the post. Above the ceiling, the problem is links, not content.

Rule of thumb: **ceiling = own authority + 25, rounded down to a multiple of 5, never below 25 or above 70.**

| Own authority (0–100 scale) | `{{MAX_DIFFICULTY}}` ceiling |
| --------------------------- | ---------------------------- |
| 0–9                         | 25                           |
| 10–14                       | 35                           |
| 15–19                       | 40                           |
| 20–29                       | 45–50                        |
| 30–44                       | 55–65                        |
| 45 and above                | 70                           |

**Worked example.** A small site at domain authority about 11 ran for months with minimum volume 30 and maximum difficulty 35. It earned tickets every week and its drafts had a realistic chance of a top-10 slot. That is where `{{MIN_VOLUME | 30}}` and `{{MAX_DIFFICULTY | 35}}` come from. Re-derive both for your site.

- Read own authority with C6 at build, and again monthly in the refresh review. Label it with the tool and date. When it crosses a row boundary, propose the new ceiling to the owner. Do not change it yourself.
- **Scales are not comparable between tools.** Ubersuggest SD, Semrush KD%, and Ahrefs KD are built differently; an Ahrefs KD of 35 is harder than an Ubersuggest SD of 35. Authority scores differ too (Ubersuggest DA, Semrush Authority Score, Ahrefs DR, DataForSEO rank on 0–1000). When you switch tools, re-read own authority in the new tool, then look at the SERPs of five keywords near the ceiling and adjust until the ceiling sits where a site your size can still win.

### SERP composition (downgrade)

Difficulty is an estimate. The SERP is evidence. Run C5 on every primary keyword that passed volume and difficulty.

- **All brands.** If every top-10 result is on a domain with authority at or above `{{SERP_BRAND_AUTHORITY | 60}}` (national brands, big marketplaces, government portals), reject the keyword for a new post, whatever its difficulty. Reason: `SERP brands`.
- **Mostly brands.** If 8 or 9 of the top 10 are at or above that authority and no result is from a site within 15 points of your own authority, set Priority to Low.
- **Small sites win.** If at least one top-10 result is from a site within 15 points of your own authority, note it. That is the best evidence a post can rank.
- **Intent mismatch.** If the top results are a different kind of page than the ticket would produce (tools, directories, product pages, a map pack, video), reject with `SERP mismatch`, unless an archetype in the brief produces that kind of page.
- Record in the ticket: who ranks (domains, not full lists), the content type that wins, small sites in the top 10 (yes or no), and the downgrade decision with its reason.
- Unknown authority never counts as small. If the tool returns no per-result authority, call a result a brand only when it is plainly a national brand, a big marketplace, or a government site; everything else is `unknown`.

## Other tools

Fill the brief's copy of this table for the tool the person uses. Endpoint names below are the tools' public API names as of 2026; their MCP connectors expose the same reports under similar names. Re-read the tool's documentation before building, and record what you actually used.

| #            | Semrush (Analytics API)                                                | Ahrefs (API v3)                                        | DataForSEO (v3)                                                                                     | Google Keyword Planner (Google Ads API)                                          |
| ------------ | ---------------------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| C1           | Pick a regional `database` (country level only)                        | `country` code (country level only)                    | `location_code` from the locations list (city level in Google Ads data; country level in Labs data) | `GeoTargetConstantService.SuggestGeoTargetConstants` (city level supported)      |
| C2           | `phrase_this` (volume `Nq`, CPC `Cp`) + `phrase_kdi` (difficulty `Kd`) | Keywords Explorer `overview` (volume, KD, CPC)         | `dataforseo_labs/google/keyword_overview/live` (volume, CPC, keyword difficulty)                    | `KeywordPlanIdeaService.GenerateKeywordHistoricalMetrics` (volume only)          |
| C3           | `phrase_related`, `phrase_fullsearch`, `phrase_questions`              | Keywords Explorer `matching-terms`, `related-terms`    | `dataforseo_labs/google/keyword_suggestions/live`, `related_keywords/live`                          | `KeywordPlanIdeaService.GenerateKeywordIdeas`                                    |
| C4           | none (use `phrase_questions`)                                          | Keywords Explorer `search-suggestions`                 | `serp/google/autocomplete/live/advanced`                                                            | none                                                                             |
| C5           | `phrase_organic` (top domains and URLs)                                | `serp-overview` (top results with DR)                  | `serp/google/organic/live/advanced`; authority per result needs `backlinks/bulk_ranks/live`         | none                                                                             |
| C6           | Authority Score (`ascore` in `backlinks_overview`)                     | Site Explorer `domain-rating` (DR)                     | `backlinks/summary/live` (`rank`, 0–1000)                                                           | none                                                                             |
| C7           | `domain_organic`, `url_organic`                                        | Site Explorer `organic-keywords` (domain or exact URL) | `dataforseo_labs/google/ranked_keywords/live` (domain or URL)                                       | none (ideas from a URL seed are not rankings)                                    |
| C8           | API units balance (`countapiunits`)                                    | `subscription-info/limits-and-usage`                   | `appendix/user_data` (money balance)                                                                | none (count calls; Google Ads API has daily operation limits)                    |
| Billing note | Per line returned: always set `display_limit`                          | Per row returned: always set `limit`                   | Per task, in money: set a per-run spend cap `{{RUN_SPEND_CAP}}` instead of a call count             | Free with an Ads account; volumes may be bucketed for accounts with little spend |

Forbidden in every tool: content or article generation, bulk exports, and any report that re-crawls or recalculates on demand. Name them in the brief once you have read the tool's price list.

For any other tool (for example OpenSEO), copy the eight rows, fill in what it offers, and mark the rest `none`. A tool that cannot do C2 in the brief's market cannot run the Gap Scan.

### When a tool lacks a metric

| Missing                          | Do this                                                                                                                                                                                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Difficulty                       | Leave it blank, labelled `n/a (<tool>)`. The SERP check becomes the difficulty gate: pass only when at least one top-10 result is from a site within 15 points of your own authority. Priority is at most Medium.                                |
| Exact volume (ranges only)       | Store the range in the ledger's `Volume Range`. Compare its **lower bound** to the minimum. Label it `range`.                                                                                                                                    |
| City-level volume                | Measure the place-modified phrase (`<topic> <place>`) at country level. Label it `country-level, place-modified`. Never prorate a country volume by population.                                                                                  |
| SERP                             | Use the agent's web search to see who ranks, without authority numbers. Mark the ticket `SERP: no authority data`, Priority at most Medium, and ask the topic approver to look before approving.                                                 |
| Per-result authority in the SERP | Classify only obvious national brands, big marketplaces, and government sites as brands. Everything else is `unknown`. Unknown never counts as small.                                                                                            |
| Own authority                    | Ask the owner for a number with its source and date. If there is none, use the 0–9 row (ceiling 25) until there is.                                                                                                                              |
| Related keywords                 | Use autocomplete (C4) plus the brief's seed lists, then C2 on each phrase you keep.                                                                                                                                                              |
| Domain and page keywords         | If a search-analytics source is connected (Google Search Console, Bing Webmaster Tools), use its queries and average positions, labelled with that source. Otherwise skip the striking-distance slot and check only the primary keyword with C2. |
| Quota endpoint                   | Count calls yourself and stop at the routine budget.                                                                                                                                                                                             |
