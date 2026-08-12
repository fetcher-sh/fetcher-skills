---
name: x-api
description: >-
  An X API alternative and Twitter API alternative on fetcher.sh — pay-per-call
  in USDC via x402, or prepaid credits with a Bearer key, no OAuth and no
  developer application. Use when the user wants to search X posts by
  keyword, hashtag, or advanced operators (from:, to:, since:, until:,
  min_faves:, filter:), scrape an X/Twitter profile by handle, pull a user's
  posts, replies, followers, or followings, fetch a single post with its
  replies or reposters, read an X List's members or posts, check trending
  topics by country, or search for X accounts by name. Also covers building an
  X data pipeline, social listening, competitor monitoring, hashtag tracking,
  or follower export without the official X API's pricing tiers or
  app-review process.
keywords:
  - x
  - twitter
  - x-api
  - twitter-api
  - x-api-alternative
  - twitter-api-alternative
  - x-data-api
  - social-media
  - social-listening
  - x402
  - ai-agent
---

# X (Twitter) Data API

A drop-in X data source for agents: search posts, resolve profiles, pull
timelines and followers, read Lists, and check trends — all as one plain HTTP
GET, paid per call. No developer account, no app review, no OAuth handshake,
and no waiting on X's own API tiers or rate-limit approvals. If your task
mentions an X search query, a handle, a post/tweet ID, or a list ID, this is
the skill.

Base URL: `https://twitter.fetcher.sh` — the API host is still named
`twitter` (it predates the rebrand and matches what agents already search
for); everything below is current X data.

Also published as [`twitter-api`](../twitter-api/SKILL.md) — identical
endpoints, indexed under the "Twitter" name too since both are still in
everyday use.

## Quick reference

| | |
| --- | --- |
| Base URL | `https://twitter.fetcher.sh` |
| Auth | `Authorization: Bearer bby_live_...` or x402 (USDC) |
| Price | $0.002–$0.005/call |
| Endpoints | 15, all `GET` |
| MCP | `https://twitter.fetcher.sh/mcp` |
| Machine-readable | `/openapi.json` · `/llms.txt` · `/skill.md` |

## Which endpoint do I need?

| I want to... | Call |
| --- | --- |
| Search posts by keyword or operator (`from:`, `since:`, `min_faves:`, ...) | `GET /api/search` |
| Search accounts by name | `GET /api/search/users` |
| Look up a profile by @handle | `GET /api/handle/{handle}` |
| Get a user's followers or followings | `GET /api/user/{id}/followers` or `/followings` |
| Get a single post by ID | `GET /api/tweet/{id}` |
| See who reposted a post | `GET /api/tweet/{id}/retweeters` |
| Read an X List's posts or members | `GET /api/list/{id}/tweets` or `/members` |
| Check trending topics for a country | `GET /api/trends` |

Full param details for every row: [`references/endpoints.md`](references/endpoints.md).

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/search?query=hello"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoints (15 — all GET, $0.005/call unless noted)

| Endpoint | Price | What it returns |
| --- | --- | --- |
| `/api/search` | $0.005 | Posts matching a query; supports X's advanced search operators |
| `/api/search/users` | $0.005 | Accounts matching a name/keyword query |
| `/api/handle/{handle}` | $0.005 | Profile by @handle |
| `/api/handle/{handle}/about` | $0.005 | Extended profile/about info by @handle |
| `/api/user/{id}` | $0.005 | Profile by numeric user ID |
| `/api/user/{id}/tweets` | $0.005 | A user's post timeline |
| `/api/user/{id}/replies` | $0.005 | A user's replies |
| `/api/user/{id}/followers` | $0.005 | A user's followers |
| `/api/user/{id}/followings` | $0.005 | Accounts a user follows |
| `/api/tweet/{id}` | **$0.002** | A single post by ID |
| `/api/tweet/{id}/replies` | $0.005 | Replies to a post |
| `/api/tweet/{id}/retweeters` | $0.005 | Accounts that reposted a post |
| `/api/list/{id}/members` | $0.005 | An X List's member accounts |
| `/api/list/{id}/tweets` | $0.005 | An X List's post feed |
| `/api/trends` | $0.005 | Trending topics for a country |

`{id}` / `{handle}` are path parameters — substitute the real value. Optional
query params (`cursor`, `sort`) paginate or reorder; only `query` (search) and
`country` (trends) are required elsewhere they appear.

## Scenarios

The query on `/api/search` goes straight to X's own search, so its operators
work as-is: `from:`, `to:`, `since:`, `until:`, `min_faves:`, `min_retweets:`,
`filter:`, `-filter:`.

**Everything from one account:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/search?query=from%3AOpenAI&sort=Latest"
```

**Between two dates:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=x402 since:2026-01-01 until:2026-02-01" -G \
  "https://twitter.fetcher.sh/api/search"
```

**Popular posts only, replies excluded:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=ai agents min_faves:500 -filter:replies" -G \
  --data-urlencode "sort=Top" \
  "https://twitter.fetcher.sh/api/search"
```

**Search accounts by name:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=climate scientist" -G \
  "https://twitter.fetcher.sh/api/search/users"
```

**Resolve a profile by handle, then pull its bio/about:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/handle/nasa"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/handle/nasa/about"
```

**A user's posts, replies, followers, or followings (by numeric ID from the
handle lookup above):**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/user/11348282/tweets"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/user/11348282/followers"
```

**A single post, its replies, and who reposted it:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/tweet/1234567890123456789"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/tweet/1234567890123456789/replies"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/tweet/1234567890123456789/retweeters"
```

**An X List's members and posts:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/list/1234567890/members"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/list/1234567890/tweets"
```

**Trending topics for a country:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://twitter.fetcher.sh/api/trends?country=United%20States"
```

## MCP

```json
{
  "mcpServers": {
    "x": {
      "url": "https://twitter.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`twitter_search`. Drop the `headers` block to pay per call with x402 instead —
see the [`fetcher` skill](../fetcher/SKILL.md) for the full flow.

## Errors

- `400` — missing/invalid parameter (message names it)
- `401` — unknown or rotated key
- `402` — payment required (x402 challenge) or `topup_required` (credits
  exhausted)
- `404` — not a priced path
- No rate limits; no refunds on upstream failures (settlement precedes
  delivery)

## Reference

- Deep dives: [`references/endpoints.md`](references/endpoints.md) (every
  param) · [`references/scenarios.md`](references/scenarios.md) (one `curl`
  per endpoint) · [`references/faq.md`](references/faq.md) ·
  [`references/comparison.md`](references/comparison.md) (vs. the official
  X API and a browser scraper)
- Task guides: [search tweets](../../task-guides/search-tweets.md) ·
  [export followers](../../task-guides/export-twitter-followers.md)
- Slash command: [`/x-search`](../../commands/x-search.md)
- Full agent setup: <https://twitter.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://twitter.fetcher.sh/openapi.json>
- Condensed catalog: <https://twitter.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://twitter.fetcher.sh>
