---
name: tiktok-profile-and-followers
description: How to look up a TikTok creator by @username and pull their posts, followers, and followings using fetcher.sh's paginated API.
license: MIT
---

# Look up a TikTok creator's profile, posts, and followers

## The task

You have a TikTok `@username` and want the profile's numeric ID, their
recent posts, and their follower/following list.

## The endpoints

```
GET https://tiktok.fetcher.sh/api/user/handle/{username}
GET https://tiktok.fetcher.sh/api/user/{id}/posts
GET https://tiktok.fetcher.sh/api/user/{id}/followers
GET https://tiktok.fetcher.sh/api/user/{id}/followings
```

| Param | Required | Notes |
| --- | --- | --- |
| `region` | no (posts only) | 2-letter country code |
| `cursor` | no | Pass back the value from a previous response to page forward |

Price: $0.004/call each. Auth: `Authorization: Bearer bby_live_...` or an
x402 payment — see the [`fetcher` skill](../skills/fetcher/SKILL.md).

## Step by step

1. Resolve the handle to a numeric ID — every other call below needs this ID,
   not the `@username`:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://tiktok.fetcher.sh/api/user/handle/khaby.lame"
   ```

2. Pull their recent posts:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://tiktok.fetcher.sh/api/user/6820094808943265798/posts"
   ```

3. Pull followers, one page at a time:

   ```bash
   curl -H "Authorization: Bearer $FETCHER_API_KEY" \
     "https://tiktok.fetcher.sh/api/user/6820094808943265798/followers"
   ```

   Take the `cursor` from the response and repeat with
   `cursor=<value>` until a page comes back without one.

4. Pull who they follow the same way, against `/followings` instead of
   `/followers`.

## Common mistakes

- Calling `/api/user/{id}/posts` etc. with the `@username` instead of the
  numeric ID from step 1 — those endpoints only accept the ID.
- Treating a paused/private account's empty follower page as an error —
  fetcher.sh returns whatever the underlying TikTok page returns; a genuinely
  empty or restricted list comes back empty, not as an HTTP error.
- Expecting a bulk "export all followers as CSV" call — build the CSV
  yourself from the paginated JSON, the same pattern as the [X/Twitter
  follower export guide](export-twitter-followers.md).

## Related

- [`references/scenarios.md`](../skills/tiktok-api/references/scenarios.md) — more worked examples for every TikTok endpoint
- [`references/faq.md`](../skills/tiktok-api/references/faq.md) — related "how do I..." questions
- [`tiktok-viral-post-search.md`](tiktok-viral-post-search.md) — find posts before you look up who made them
