---
name: instagram-profile-lookup
description: How to look up an Instagram profile by @handle and pull its posts, reels, and stories using fetcher.sh's API, no Meta developer account or connected account required.
license: MIT
---

# Look up an Instagram profile, posts, reels, and stories

## The task

You have an Instagram `@handle` and want the full profile plus their recent
posts, reels, and currently active stories — for a public account you don't
own or manage.

## The endpoints

```
GET https://instagram.fetcher.sh/api/user/handle/{handle}
GET https://instagram.fetcher.sh/api/user/{id}/posts
GET https://instagram.fetcher.sh/api/user/{id}/reels
GET https://instagram.fetcher.sh/api/user/{id}/stories
```

| Param | Required | Notes |
| --- | --- | --- |
| `cursor` | no (posts/reels only) | Pass back the value from a previous response to page forward |

Price: $0.004/call each. Auth: `Authorization: Bearer bby_live_...` or an
x402 payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

## Step by step

1. Resolve the handle to the full profile and its numeric ID in one call —
   this is the hero endpoint:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://instagram.fetcher.sh/api/user/handle/natgeo"
   ```

2. Pull their posts, paginating with `cursor` as needed:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://instagram.fetcher.sh/api/user/787132/posts"
   ```

3. Pull their reels the same way, against `/reels` instead of `/posts`.
4. Pull their currently active stories:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://instagram.fetcher.sh/api/user/787132/stories"
   ```

   This returns only what's live right now — there's no stories archive or
   history endpoint, matching what Instagram itself exposes to non-owners.

## Common mistakes

- Expecting `/stories` to return past stories once they've expired — it
  can't; that data isn't available from the underlying platform either.
- Using `/api/userid/{handle}` (ID-only) when you actually need the full
  profile — use `/api/user/handle/{handle}` instead and skip the extra call.
- Assuming posts and tagged posts are the same thing — `/api/user/{id}/posts`
  is what they posted; `/api/user/{id}/posts/tagged` is what others posted
  and tagged them in.

## Related

- [`references/scenarios.md`](../skills/instagram-api/references/scenarios.md) — more worked examples for every Instagram endpoint
- [`references/faq.md`](../skills/instagram-api/references/faq.md) — related "how do I..." questions
- [`instagram-hashtag-and-location-monitoring.md`](instagram-hashtag-and-location-monitoring.md) — the discovery-side counterpart to this profile lookup
