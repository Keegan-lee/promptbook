# CP · Analytics Pull

| Setting | Value                                                                                                                                                                                                             |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Trigger | Daily at `{{ANALYTICS_TIME \| 06:00}}` in `{{TIME_ZONE}}`                                                                                                                                                         |
| Reads   | Platform Posts (Posted); the distributor's analytics, or each platform's native analytics                                                                                                                         |
| Writes  | Platform Posts → Views, Likes, Comments, Shares, Saves, Clicks, Avg View %, Engagement Rate, Engagement Score (1–10), Metrics Updated At, Notes; and, in Live, Approval and Review Notes for back-to-review posts |
| Modes   | Paused; Dry Run (metrics and scores, back-to-review listed only); Live (also moves back-to-review posts)                                                                                                          |

## Purpose

Bring each post's numbers home every day, score each post against its own channel's history, and pull a weak piece's remaining posts back for a second look before they go out.

## Rules

- **Write only what the source returned.** A metric the platform does not report stays blank. Never estimate one.
- **People never edit Engagement Score.** This job owns it.
- Back to review is the only Approval change this job may make, and only from Approved to Needs Review on posts that have not been sent.

## Steps

1. Read the Settings row. Paused: stop.
2. Find posts with Upload Status = Posted and Published At within the last `{{ANALYTICS_WINDOW_DAYS | 30}}` days. Batch: all of them, fetched in pages.
3. For each, fetch metrics by its request id, or, for a post someone posted by hand, by the platform post id in its Platform Post URL where the source supports that. No match: leave it blank. Write Views, Likes, Comments, Shares, Saves, Clicks, Avg View % (0–100) as returned, and Metrics Updated At = now.
4. **Engagement Rate**, when Views > 0:

   ```
   {{ENGAGEMENT_FORMULA | (Likes + 2×Comments + 3×Shares + 2×Saves + Clicks) ÷ Views}}
   ```

   A blank metric counts as 0 in the rate. Views of 0 or blank: leave the rate blank.

5. **Engagement Score (1–10)** for a post that is **eligible**: Views ≥ `{{SCORE_MIN_VIEWS | 50}}` and Published At at least `{{SCORE_MIN_HOURS | 72}}` hours ago.
   1. **History:** every eligible post on the same Platform with an Engagement Rate, Published At within the last `{{SCORE_HISTORY_DAYS | 90}}` days, this post included.
   2. Fewer than `{{SCORE_MIN_HISTORY | 5}}` posts in the history: leave the score blank and add a Notes line `Score waits: <n> scored posts on this channel` (once). One channel's early weeks cannot define typical.
   3. **Ratio** = this post's rate ÷ the median rate of the history.
   4. Map the ratio to a score, so 5 means typical for this channel:

      | Ratio         | Score                      |
      | ------------- | -------------------------- |
      | under 0.25    | 1 — Very low               |
      | 0.25 to 0.40  | 2 — Low                    |
      | 0.40 to 0.60  | 3 — Below average          |
      | 0.60 to 0.85  | 4 — Slightly below average |
      | 0.85 to 1.15  | 5 — Average                |
      | 1.15 to 1.40  | 6 — Slightly above average |
      | 1.40 to 1.75  | 7 — Good                   |
      | 1.75 to 2.25  | 8 — Strong                 |
      | 2.25 to 3.00  | 9 — Excellent              |
      | 3.00 and over | 10 — Exceptional           |

      Each band includes its lower bound. Write the option label exactly as it appears in the field's choices.

6. **Back to review.** For each Content row that has at least two scored posts, all scored `{{BACK_TO_REVIEW_MAX_SCORE | 3}}` or lower:
   - find its other posts with Approval = Approved and Upload Status = Pending;
   - **Dry Run:** list their ids in the Last Run Summary. Change nothing.
   - **Live:** set Approval = Needs Review, and add a Review Notes line `BACK TO REVIEW <date> (CP · Analytics Pull): the sibling posts scored <scores>. Recheck the angle before it goes out.` The stamp automation clears the stamps, so the post needs a fresh human approval.
   - Each post moves at most once: skip a post whose Review Notes already starts with `BACK TO REVIEW`.
7. Write the Last Run Summary: posts updated, posts scored, posts waiting for history, back-to-review moves (ids), sources that failed.

If the analytics source is unreachable, leave every metric as it was and name the source in the summary. Absence of access is not a zero.

## n8n notes

- Upload-Post exposes per-post analytics by request id (connector: `get_post_analytics`, which fetches live from the platform; `get_cached_post_analytics` returns the last copy without a new platform call) and per-profile analytics. Use the per-post call, at most once per post per day. Each platform reports its own shape: map only the counters it returns. Platforms that report no saves or clicks leave those fields blank.
- Without a workflow engine, an agent routine runs this spec with the distributor's connector and only its read tools (analytics, status, history).
- Compute the median and the bands in one Code node per run, grouping posts by Platform.
- Write in batches of 10 records.

## Test

Run it once in Dry Run after the first live posts are a few days old. Compare three posts' numbers with what the platform shows. Check that no post has a score before its channel has five eligible posts.
