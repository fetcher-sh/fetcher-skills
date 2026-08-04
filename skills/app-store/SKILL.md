---
name: app-store
description: >-
  An Apple App Store API alternative on fetcher.sh — pay-per-call in USDC via
  x402, or prepaid credits with a Bearer key, no Apple Developer Program
  membership. Use when the user wants to search iOS apps by keyword in any
  country's storefront, fetch an app's or app bundle's full details, reviews
  sorted by recent/helpful, apps similar to a given app, search or fetch app
  bundles, or fetch a developer's app catalog. Also covers iOS app store
  optimization (ASO) research, competitor app monitoring, localized
  storefront comparison, and app discovery without Apple Developer Program
  access.
keywords:
  - app-store
  - apple-app-store
  - ios-apps
  - app-store-api
  - app-store-optimization
  - aso
  - itunes-api
  - x402
  - ai-agent
---

# Apple App Store API

App Store app and bundle search, details, reviews, similar-apps, and
developer catalogs across any country storefront — one plain HTTP GET per
call, paid as you go. No Apple Developer Program membership, no App Store
Connect access, no scraping the storefront HTML.

Base URL: `https://appstore.fetcher.sh`

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://appstore.fetcher.sh/api/apps?term=meditation"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoints (9 — all GET, $0.003/call)

| Endpoint | What it returns |
| --- | --- |
| `/api/apps` | Apps matching a keyword; page and country storefront filters |
| `/api/apps/{appId}` | An app's full details |
| `/api/apps/{appId}/reviews` | An app's reviews |
| `/api/apps/{appId}/similar` | Apps similar to a given app |
| `/api/bundles` | App bundles matching a keyword |
| `/api/bundles/{bundleId}` | A bundle's full details |
| `/api/bundles/{bundleId}/reviews` | A bundle's reviews |
| `/api/bundles/{bundleId}/similar` | Bundles similar to a given bundle |
| `/api/developers/{developerId}` | A developer's full app catalog |

Required on `/api/apps` and `/api/bundles`: `term`. Optional everywhere:
`country` (storefront, e.g. `us`, `jp`, `de`), `lang`, `page`. Reviews
support `sort` (`recent`, `helpful`).

## Scenarios

**US storefront search:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "term=habit tracker" -G \
  --data-urlencode "country=us" \
  "https://appstore.fetcher.sh/api/apps"
```

**Japanese storefront search:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "term=camera" -G \
  --data-urlencode "country=jp" \
  "https://appstore.fetcher.sh/api/apps"
```

**An app's full details, then reviews sorted by helpfulness:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "country=us" -G \
  "https://appstore.fetcher.sh/api/apps/324684580"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "country=us" -G \
  --data-urlencode "sort=helpful" \
  "https://appstore.fetcher.sh/api/apps/324684580/reviews"
```

**Apps similar to a given app:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "country=us" -G \
  "https://appstore.fetcher.sh/api/apps/324684580/similar"
```

**Search and fetch app bundles:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "term=photo editing suite" -G \
  "https://appstore.fetcher.sh/api/bundles"

curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "country=us" -G \
  "https://appstore.fetcher.sh/api/bundles/1234567890"
```

**A developer's full app catalog:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://appstore.fetcher.sh/api/developers/284882218"
```

## MCP

```json
{
  "mcpServers": {
    "app-store": {
      "url": "https://appstore.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`appstore_apps`. Drop the `headers` block to pay per call with x402 instead —
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

- Full agent setup: <https://appstore.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://appstore.fetcher.sh/openapi.json>
- Condensed catalog: <https://appstore.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://appstore.fetcher.sh>
