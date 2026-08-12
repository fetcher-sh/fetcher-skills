---
name: export-twitter-followers
description: How to pull a full follower or following list from an X (Twitter) account via fetcher.sh's paginated API, and turn it into a CSV yourself — honest about what is and isn't automated.
license: MIT
---

# Export an X (Twitter) account's followers

## The task

You want the full follower list (or following list) for an X/Twitter
account — for a CSV export, a spreadsheet, or further processing.

## What fetcher.sh actually does here

There's no server-side "export" job and no CSV output — `twitter.fetcher.sh`
returns JSON, one page of followers per call, and you paginate through all of
it yourself. If you want a CSV at the end, you write that conversion step; it
isn't part of the API.

## The endpoints

```
GET https://twitter.fetcher.sh/api/handle/{handle}
GET https://twitter.fetcher.sh/api/user/{id}/followers
GET https://twitter.fetcher.sh/api/user/{id}/followings
```

| Param | Required | Notes |
| --- | --- | --- |
| `cursor` | no | Pass back the value from a previous response to page forward |

Price: $0.005/call each. Auth: `Authorization: Bearer bby_live_...` or an
x402 payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

## Step by step

1. Resolve the handle to a numeric ID (you need this for every step after):

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://twitter.fetcher.sh/api/handle/nasa"
   ```

2. Pull the first page of followers:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://twitter.fetcher.sh/api/user/11348282/followers"
   ```

3. Take the `cursor` from that response and request the next page:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     --data-urlencode "cursor=<cursor from step 2>" -G \
     "https://twitter.fetcher.sh/api/user/11348282/followers"
   ```

4. Repeat until a response comes back without a `cursor` — that's the last
   page.
5. Append each page's follower records to a list, then write that list to a
   `.csv` with whatever tool you're already using (`csv` in Python,
   `papaparse` in JS, `jq` for a quick shell pipeline, etc.). Fetcher.sh's
   job ends at step 4.

## Cost and scale math

Each page is one $0.005 call. A 50,000-follower account paginated in pages of
a few hundred is on the order of a few dollars total, not per-follower
pricing — check the actual page size in your first response before
estimating a large account's cost.

## Common mistakes

- Expecting a `?format=csv` or `?export=true` param — it doesn't exist.
- Treating the follower endpoint as real-time/streaming — every call is a
  fresh on-demand read; there's no persistent subscription.
- Using the `@handle` instead of the numeric ID on the `/followers` and
  `/followings` endpoints — those two specifically require the ID from step 1.

## Related

- [`references/endpoints.md`](../skills/twitter-api/references/endpoints.md) — full parameter reference
- [`references/faq.md`](../skills/twitter-api/references/faq.md) — related "how do I..." questions
- [`tiktok-profile-and-followers.md`](tiktok-profile-and-followers.md) — the same pattern on TikTok
