---
name: youtube-api
description: >-
  A YouTube API alternative on fetcher.sh — pay-per-call in USDC via x402, or
  prepaid credits with a Bearer key, no OAuth and no daily quota. Use when
  the user wants to search YouTube videos with filters for upload date,
  duration, or sort order, search channels or playlists, look up a channel by
  ID, @handle, or custom URL path, list a channel's videos, shorts, or live
  streams, fetch a video's or short's details and comments, list a
  playlist's videos, pull posts under a hashtag, or check trending videos by
  region. Also covers YouTube data pipelines, channel monitoring, video
  analytics input, and trend tracking without the official YouTube Data API's
  daily quota limits.
keywords:
  - youtube
  - youtube-api
  - youtube-api-alternative
  - youtube-data
  - video-search
  - x402
  - ai-agent
---

# YouTube API

YouTube data on demand: video/channel/playlist search with real filters
(upload date, duration, sort order), channel and video lookups, videos,
shorts, live streams, playlist contents, comments, hashtag feeds, and trending
— one plain HTTP GET per call, paid as you go. No developer console project,
no OAuth consent screen, no daily quota.

Base URL: `https://youtube.fetcher.sh`

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/search/video?query=hello"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoints (15 — all GET, $0.005/call)

| Endpoint | What it returns |
| --- | --- |
| `/api/search/video` | Videos matching a query; upload date, duration, sort filters |
| `/api/search/channel` | Channels matching a query |
| `/api/search/playlist` | Playlists matching a query |
| `/api/channel/{id}` | Channel details by ID |
| `/api/channel/handle/{handle}` | Channel details by @handle |
| `/api/channel/path` | Channel details by custom URL path |
| `/api/channel/{id}/videos` | A channel's videos |
| `/api/channel/{id}/shorts` | A channel's shorts |
| `/api/channel/{id}/live-streams` | A channel's live streams |
| `/api/video/{id}` | Video (or short) details |
| `/api/video/{id}/comments` | A video's comments |
| `/api/shorts/{id}` | Short details |
| `/api/playlist/{id}/videos` | A playlist's videos |
| `/api/hashtag/{tag}` | Videos under a hashtag |
| `/api/trending` | Trending videos, optionally by region |

`{id}` / `{handle}` / `{tag}` are path parameters. Optional `cursor` /
`lang` / `geo` refine results; `query` (search) and `channelPath` (path
lookup) are required where they appear.

## Scenarios

**Uploaded this week:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=ai agents" -G \
  --data-urlencode "uploadDate=week" \
  "https://youtube.fetcher.sh/api/search/video"
```

**Most viewed, long-form only:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=x402 protocol" -G \
  --data-urlencode "sortBy=view_count" \
  --data-urlencode "duration=long" \
  "https://youtube.fetcher.sh/api/search/video"
```

**Newest first:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=stablecoin payments" -G \
  --data-urlencode "sortBy=upload_date" \
  "https://youtube.fetcher.sh/api/search/video"
```

Other `uploadDate` values: `hour`, `today`, `month`, `year`. Other `duration`
values: `short`, `medium`. Other `sortBy` values: `relevance`, `rating`.

**Search channels or playlists:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=lofi hip hop" -G \
  "https://youtube.fetcher.sh/api/search/channel"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=lofi hip hop" -G \
  "https://youtube.fetcher.sh/api/search/playlist"
```

**A channel by ID, handle, or custom path — then its videos, shorts, and live
streams:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/channel/handle/mkbhd"

curl -H "Authorization: Bearer $FETCHER_API_KEY" -G \
  --data-urlencode "channelPath=@mkbhd" \
  "https://youtube.fetcher.sh/api/channel/path"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/channel/UCBJycsmduvYEL83R_U4JriQ/videos"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/channel/UCBJycsmduvYEL83R_U4JriQ/shorts"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/channel/UCBJycsmduvYEL83R_U4JriQ/live-streams"
```

**A video's details and comments, or a short's details:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/video/dQw4w9WgXcQ"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=top" -G \
  "https://youtube.fetcher.sh/api/video/dQw4w9WgXcQ/comments"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/shorts/abc123XYZ90"
```

**A playlist's videos, videos under a hashtag, and trending (by region):**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/playlist/PLFgquLnL59alCl_2TQvOiD5Vgm1hCaGSI/videos"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://youtube.fetcher.sh/api/hashtag/shorts"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "geo=US" -G \
  "https://youtube.fetcher.sh/api/trending"
```

## MCP

```json
{
  "mcpServers": {
    "youtube": {
      "url": "https://youtube.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`youtube_search_video`. Drop the `headers` block to pay per call with x402
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

- Full agent setup: <https://youtube.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://youtube.fetcher.sh/openapi.json>
- Condensed catalog: <https://youtube.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://youtube.fetcher.sh>
