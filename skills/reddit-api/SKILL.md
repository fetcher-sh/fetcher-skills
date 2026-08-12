---
name: reddit-api
description: >-
  A Reddit API alternative on fetcher.sh — pay-per-call in USDC via x402, or
  prepaid credits with a Bearer key, no OAuth app registration. Use when the
  user wants to search Reddit posts across every subreddit by keyword and
  sort by top, hot, new, or most-discussed, search subreddits or users by
  keyword, fetch a subreddit's info or its hot/new/top post feed, fetch a
  single post with its comment tree and comment replies, pull the sitewide
  best/hot/new/top feeds, or fetch a user's profile, posts, and comments.
  Also covers Reddit sentiment analysis input, subreddit monitoring, keyword
  tracking, and Reddit data pipelines without Reddit's own API app
  registration or rate-limit tiers.
keywords:
  - reddit
  - reddit-api
  - reddit-api-alternative
  - subreddit
  - social-listening
  - x402
  - ai-agent
---

# Reddit API

Reddit data on demand: cross-subreddit post search, subreddit and user search,
subreddit info and feeds, single posts with full comment trees, sitewide
best/hot/new/top feeds, and user profiles/posts/comments — one plain HTTP GET
per call, paid as you go. No OAuth app registration, no client ID/secret pair,
no Reddit API rate-limit tier.

Base URL: `https://reddit.fetcher.sh`

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/search/post?keyword=hello"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoints (15 — all GET, $0.002/call)

| Endpoint | What it returns |
| --- | --- |
| `/api/search/post` | Posts across every subreddit matching a keyword |
| `/api/search/subreddit` | Subreddits matching a keyword |
| `/api/search/user` | Users matching a keyword |
| `/api/subreddit/{name}` | A subreddit's info |
| `/api/subreddit/{name}/posts` | A subreddit's post feed |
| `/api/post/{id}` | A single post |
| `/api/post/{id}/comments` | A post's comment tree |
| `/api/post/{id}/comments/{commentId}/replies` | Replies to a comment |
| `/api/posts/hot` | Sitewide hot feed |
| `/api/posts/new` | Sitewide new feed |
| `/api/posts/top` | Sitewide top feed |
| `/api/posts/best` | Sitewide best feed |
| `/api/user/{username}` | A user's profile |
| `/api/user/{username}/posts` | A user's posts |
| `/api/user/{username}/comments` | A user's comments |

`{id}` / `{name}` / `{username}` are path parameters. Optional `sort` /
`cursor` reorder and paginate; `keyword` (search) is required where it
appears.

## Scenarios

**Top posts about a topic, across every subreddit:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "keyword=ai agents" -G \
  --data-urlencode "sort=top" \
  "https://reddit.fetcher.sh/api/search/post"
```

**Most discussed (highest comment count):**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "keyword=stablecoin payments" -G \
  --data-urlencode "sort=comments" \
  "https://reddit.fetcher.sh/api/search/post"
```

**Newest first:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "keyword=x402" -G \
  --data-urlencode "sort=new" \
  "https://reddit.fetcher.sh/api/search/post"
```

Other `sort` values for `/api/search/post`: `relevance`, `hot`.

**Search subreddits or users by keyword:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "keyword=machine learning" -G \
  "https://reddit.fetcher.sh/api/search/subreddit"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "keyword=spez" -G \
  "https://reddit.fetcher.sh/api/search/user"
```

**A subreddit's info, then its hot/new/top feed:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/subreddit/MachineLearning"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=top" -G \
  "https://reddit.fetcher.sh/api/subreddit/MachineLearning/posts"
```

**A single post, its comment tree, and replies to one comment:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/post/1abcde2"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=top" -G \
  "https://reddit.fetcher.sh/api/post/1abcde2/comments"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/post/1abcde2/comments/fghij3k/replies"
```

**Sitewide feeds:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/posts/hot"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/posts/best"
```

**A user's profile, posts, and comments:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://reddit.fetcher.sh/api/user/spez"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=new" -G \
  "https://reddit.fetcher.sh/api/user/spez/posts"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=top" -G \
  "https://reddit.fetcher.sh/api/user/spez/comments"
```

## MCP

```json
{
  "mcpServers": {
    "reddit": {
      "url": "https://reddit.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`reddit_search_post`. Drop the `headers` block to pay per call with x402
instead — see the [`fetcher` skill](../fetcher/SKILL.md) for the full flow.

## Errors

- `400` — missing/invalid parameter (message names it)
- `401` — unknown or rotated key
- `402` — payment required (x402 challenge) or `topup_required` (credits
  exhausted)
- `404` — not a priced path
- No rate limits; no refunds on upstream failures (settlement precedes
  delivery)

## Reference

- Full agent setup: <https://reddit.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://reddit.fetcher.sh/openapi.json>
- Condensed catalog: <https://reddit.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://reddit.fetcher.sh>
