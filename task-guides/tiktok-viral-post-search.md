---
name: tiktok-viral-post-search
description: How to find viral or trending TikTok posts for a keyword using sortType and dateRange on fetcher.sh's post search endpoint — no developer account required.
license: MIT
---

# Find viral TikTok posts for a keyword

## The task

You want to know what's currently going viral on TikTok around a topic —
most-liked posts, restricted to a recent window — without a TikTok developer
account or Research API access.

## The endpoint

```
GET https://tiktok.fetcher.sh/api/post/search
```

| Param | Required | Notes |
| --- | --- | --- |
| `keyword` | yes | Search term |
| `sortType` | no | `RELEVANCE` (default), `MOST_LIKED`, `DATE_POSTED` |
| `dateRange` | no | `ALL_TIME` (default), `YESTERDAY`, `THIS_WEEK`, `THIS_MONTH`, `LAST_THREE_MONTHS`, `LAST_SIX_MONTHS` |
| `region` | no | 2-letter country code |
| `cursor` | no | Pass back the value from a previous response to page forward |

Price: $0.004/call. Auth: `Authorization: Bearer bby_live_...` or an x402
payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

## Step by step

1. Pick the recency window that matches "viral right now" for your purpose —
   `THIS_WEEK` is a reasonable default, `YESTERDAY` for same-day virality.
2. Combine that with `sortType=MOST_LIKED`:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     --data-urlencode "keyword=matcha latte" -G \
     --data-urlencode "sortType=MOST_LIKED" \
     --data-urlencode "dateRange=THIS_WEEK" \
     "https://tiktok.fetcher.sh/api/post/search"
   ```

3. Optionally scope to a country with `region=US` (or another 2-letter code)
   if you only care about one market's trend.
4. Page forward with `cursor` if you need more than one page of results.

## Reading the ranking correctly

`sortType=MOST_LIKED` ranks within whatever `dateRange` you set — it does not
search all-time and then filter by date after the fact. Narrowing the date
range first, then sorting by likes, is what actually surfaces "viral this
week" instead of "the all-time biggest hits that happen to be recent."

## Common mistakes

- Leaving `dateRange` at its default `ALL_TIME` while expecting "trending
  now" results — you'll get all-time top posts instead.
- Assuming there's a dedicated `/trending` endpoint on this host — there
  isn't; `sortType` + `dateRange` on `/api/post/search` is how you get
  the same effect.
- Expecting a push notification when a post starts trending — there's no
  webhook; re-run the search on a schedule if you need ongoing monitoring.

## Related

- [`references/scenarios.md`](../skills/tiktok-api/references/scenarios.md) — more worked examples for every TikTok endpoint
- [`references/faq.md`](../skills/tiktok-api/references/faq.md) — related "how do I..." questions
- [`tiktok-profile-and-followers.md`](tiktok-profile-and-followers.md) — look up the creators behind the posts you find
