---
name: google-maps
description: >-
  A Google Maps API alternative and Google Places API alternative on
  fetcher.sh — pay-per-call in USDC via x402, or prepaid credits with a
  Bearer key, no Google Cloud billing account. Use when the user wants to
  search places by text query or anchor the search to a latitude/longitude
  coordinate, fetch a place's full details by its feature ID (fid), pull a
  place's reviews sorted by relevance/newest/rating, or look up a single
  review by ID. Also covers local business discovery, points-of-interest
  data, store-locator input, and review monitoring without Google Cloud
  Platform's API key setup, billing, and per-request pricing.
keywords:
  - google-maps
  - google-maps-api
  - google-places-api
  - places-api-alternative
  - local-business-data
  - poi-data
  - x402
  - ai-agent
---

# Google Maps API

Place search, place details, and reviews as clean JSON — one plain HTTP GET
per call, paid as you go. No Google Cloud project, no API key restriction
rules, no per-request Places API billing.

Base URL: `https://google-maps.fetcher.sh`

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://google-maps.fetcher.sh/api/place/search?query=coffee"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoints (4 — all GET, $0.005/call)

| Endpoint | What it returns |
| --- | --- |
| `/api/place/search` | Places matching a text query, optionally anchored to a coordinate |
| `/api/place/{fid}` | A place's full details by feature ID |
| `/api/place/{fid}/reviews` | A place's reviews |
| `/api/review/{id}` | A single review by ID |

Required on search: `query`. Optional: `languageCode`, `countryCode`,
`latitude`, `longitude`, `offset`, `zoom` (search radius/precision). Reviews
support `sort` (`relevant`, `newest`, `highest`, `lowest`) and `cursor`.

## Scenarios

**Plain text search:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=ramen in brooklyn" -G \
  "https://google-maps.fetcher.sh/api/place/search"
```

**Anchor the search to a coordinate (for "near me" style queries):**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=coffee" -G \
  --data-urlencode "latitude=40.7128" \
  --data-urlencode "longitude=-74.0060" \
  "https://google-maps.fetcher.sh/api/place/search"
```

**A place's details, then its reviews sorted by newest:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://google-maps.fetcher.sh/api/place/0x89c259af336b3341%3A0x8c584d6dfe89aa89"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "sort=newest" -G \
  "https://google-maps.fetcher.sh/api/place/0x89c259af336b3341%3A0x8c584d6dfe89aa89/reviews"
```

**A single review by ID:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://google-maps.fetcher.sh/api/review/ChdDSUhNMG9nS0VJQ0FnSURDeGRXQnl3RRAB"
```

**Localized results (French language, Canadian region):**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=boulangerie" -G \
  --data-urlencode "languageCode=fr" \
  --data-urlencode "countryCode=CA" \
  "https://google-maps.fetcher.sh/api/place/search"
```

## MCP

```json
{
  "mcpServers": {
    "google-maps": {
      "url": "https://google-maps.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`google_maps_place_search`. Drop the `headers` block to pay per call with
x402 instead — see the [`fetcher` skill](../fetcher/SKILL.md) for the full
flow.

## Errors

- `400` — missing/invalid parameter (message names it)
- `401` — unknown or rotated key
- `402` — payment required (x402 challenge) or `topup_required` (credits
  exhausted)
- `404` — not a priced path
- No rate limits; no refunds on upstream failures (settlement precedes
  delivery)

## Reference

- Full agent setup: <https://google-maps.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://google-maps.fetcher.sh/openapi.json>
- Condensed catalog: <https://google-maps.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://google-maps.fetcher.sh>
