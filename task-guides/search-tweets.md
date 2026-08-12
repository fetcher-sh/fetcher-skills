---
name: search-tweets
description: How to search tweets/posts on X (Twitter) by keyword, account, date range, and engagement threshold using fetcher.sh's pay-per-call API — no developer account or OAuth required.
license: MIT
---

# Search tweets on X (Twitter) without a developer account

## The task

You need to find tweets/posts matching a keyword, from a specific account,
within a date range, or above an engagement threshold — without setting up
an X developer account, OAuth app, or paying for a monthly API tier.

## The endpoint

```
GET https://twitter.fetcher.sh/api/search
```

| Param | Required | Notes |
| --- | --- | --- |
| `query` | yes | Full X advanced-search string |
| `sort` | no | `Latest` or `Top` (default is X's own relevance ranking) |
| `cursor` | no | Pass back the value from a previous response to page forward |

Price: $0.005/call. Auth: `Authorization: Bearer bby_live_...` (prepaid
credits) or an x402 payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

`query` is passed straight through to X's own search parser, so every
operator X's search bar supports works unmodified:

| Operator | Effect |
| --- | --- |
| `from:handle` | Only tweets from that account |
| `to:handle` | Only replies directed at that account |
| `since:YYYY-MM-DD` / `until:YYYY-MM-DD` | Date range |
| `min_faves:N` / `min_retweets:N` | Engagement threshold |
| `filter:replies` / `-filter:replies` | Include or exclude replies |
| `filter:links` | Only tweets containing a link |

## Step by step

1. Build your query string by combining operators, e.g.
   `ai agents min_faves:200 since:2026-01-01 -filter:replies`.
2. Call the endpoint:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     --data-urlencode "query=ai agents min_faves:200 since:2026-01-01 -filter:replies" -G \
     --data-urlencode "sort=Top" \
     "https://twitter.fetcher.sh/api/search"
   ```

3. If you need more results than one page returns, take the `cursor` from
   the response and pass it back as `cursor=` on the next call. Repeat until
   the response stops returning a cursor.

## Common mistakes

- Splitting the query into separate `startDate`/`endDate`/`account` params —
  there aren't any; everything goes in one `query` string using X's own
  operator syntax.
- Assuming there's a full-archive guarantee — X's search index itself, not
  fetcher.sh, decides how far back results go.
- Constructing or decoding the `cursor` value — treat it as opaque and only
  ever pass back exactly what you received.

## Related

- [`references/scenarios.md`](../skills/twitter-api/references/scenarios.md) — more worked examples for every Twitter/X endpoint
- [`references/faq.md`](../skills/twitter-api/references/faq.md) — related "how do I..." questions
- [`export-twitter-followers.md`](export-twitter-followers.md) — the equivalent guide for follower lists instead of search
