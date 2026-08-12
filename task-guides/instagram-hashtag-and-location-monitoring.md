---
name: instagram-hashtag-and-location-monitoring
description: How to track new Instagram posts under a hashtag or location by polling fetcher.sh's API on a schedule — honest that this is polling, not a push/webhook subscription.
license: MIT
---

# Monitor an Instagram hashtag or location for new posts

## The task

You want to keep tabs on new posts under a hashtag (e.g. a campaign tag) or
at a location (e.g. a venue or city), for social listening or moderation.

## What fetcher.sh actually does here

There's no webhook, subscription, or streaming endpoint on any fetcher.sh
host — this is a read-only, on-demand API. "Monitoring" means you call the
same endpoint on a schedule yourself (a cron job, a scheduled function, a
loop with a `sleep`) and diff the new response against what you saw last
time.

## The endpoints

```
GET https://instagram.fetcher.sh/api/hashtag/{name}/posts
GET https://instagram.fetcher.sh/api/location/{id}/posts
```

| Param | Required | Notes |
| --- | --- | --- |
| `cursor` | no | Pass back the value from a previous response to page forward |
| `page` | no | Alternate/legacy pagination param some responses use instead of `cursor` |
| `tab` | no, location only | Selects which content section to read |

Price: $0.004/call each. Auth: `Authorization: Bearer bby_live_...` or an
x402 payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

## Step by step

1. Pull the current first page for your hashtag (no `#`) or location ID:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://instagram.fetcher.sh/api/hashtag/sunsetphotography/posts"
   ```

2. Record the set of post IDs you got back.
3. On your own schedule (every few minutes, hourly — whatever fits your
   budget and freshness need), repeat the same call.
4. Diff the new post IDs against the set from step 2; anything new is a new
   post since your last check. Update your stored set and repeat.

## Choosing a poll interval

Every call costs money, so the interval is a budget decision, not a
technical limit: polling every minute costs 60x what polling every hour
costs. Pick the loosest interval that still meets your freshness need —
hourly is plenty for most brand-monitoring use cases; minute-level polling is
usually only worth it for time-sensitive moderation.

## Common mistakes

- Building this against a webhook URL fetcher.sh would call — that
  capability doesn't exist; you must poll.
- Polling faster than you need "just in case" — it only adds cost, not
  freshness, once you're already below the latency of the underlying page.
- Forgetting hashtags are case-insensitive and `#`-free in the path — pass
  `sunsetphotography`, not `#SunsetPhotography`.

## Related

- [`references/scenarios.md`](../skills/instagram-api/references/scenarios.md) — more worked examples for every Instagram endpoint
- [`references/faq.md`](../skills/instagram-api/references/faq.md) — related "how do I..." questions
- [`instagram-profile-lookup.md`](instagram-profile-lookup.md) — look up the accounts behind the posts you find
