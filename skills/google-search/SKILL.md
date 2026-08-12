---
name: google-search
description: >-
  A Google Search API alternative and SERP API alternative on fetcher.sh —
  pay-per-call in USDC via x402, or prepaid credits with a Bearer key, no
  API key application. Use when the user wants programmatic Google search
  results as clean JSON, including Google's own operators — site:, filetype:,
  intitle:, and quoted exact phrases — plus pagination, language (hl), and
  country/region scoping. Also covers rank tracking input, competitive
  research, and search-result monitoring without Google's own Custom Search
  API quota and billing.
keywords:
  - google-search
  - serp-api
  - serp-api-alternative
  - google-search-api
  - web-search
  - web-search-api
  - x402
  - ai-agent
---

# Google Search API

Google's own search results as clean JSON — one plain HTTP GET per call, paid
as you go. Google's search operators pass straight through: `site:`,
`filetype:`, `intitle:`, and quoted exact phrases all work exactly as they do
in the browser. No Custom Search Engine setup, no per-day query quota, no
billing account.

Base URL: `https://google.fetcher.sh`

## Authentication

Two ways to pay, same data — full mechanics in the [`fetcher`
skill](../fetcher/SKILL.md):

```bash
# 1. Prepaid credits (recommended — get a key at https://fetcher.sh/topup
#    or via POST /api/credits/topup, see the fetcher skill)
export FETCHER_API_KEY="bby_live_xxxxxxxxxxxx"
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  "https://google.fetcher.sh/api/search?query=hello"

# 2. x402 pay-per-call — omit the header; a GET with no payment returns 402
#    with machine-readable payment requirements (USDC on Base, Polygon,
#    Arbitrum, Monad, or Solana). @x402/fetch signs and retries automatically.
```

Every response is `{ "status": number, "message": string, "data": ... }`; the
HTTP status mirrors `status`.

## Endpoint ($0.005/call)

| Endpoint | What it returns |
| --- | --- |
| `/api/search` | Google web search results for a query |

Required: `query`. Optional: `safe` (SafeSearch), `page` (pagination), `hl`
(UI language, e.g. `en`), `country` (region scope), `count` (results per
page — currently `10`), `noEncode` (skip query re-encoding for pre-formatted
operator strings).

## Scenarios

**Restrict to one site:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=site:github.com x402 payments" -G \
  "https://google.fetcher.sh/api/search"
```

**PDFs only:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=agentic commerce filetype:pdf" -G \
  "https://google.fetcher.sh/api/search"
```

**Exact phrase:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode 'query="pay per call api"' -G \
  "https://google.fetcher.sh/api/search"
```

**Localized results (UK, French UI) and pagination:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=agentic commerce" -G \
  --data-urlencode "country=GB" \
  --data-urlencode "hl=fr" \
  --data-urlencode "page=2" \
  "https://google.fetcher.sh/api/search"
```

**Combine operators — title match plus site restriction:**

```bash
curl -H "Authorization: Bearer $FETCHER_API_KEY" \
  --data-urlencode "query=intitle:pricing site:x402.org" -G \
  "https://google.fetcher.sh/api/search"
```

## MCP

```json
{
  "mcpServers": {
    "google-search": {
      "url": "https://google.fetcher.sh/mcp",
      "headers": { "Authorization": "Bearer bby_live_..." }
    }
  }
}
```

Free: `search_endpoints`, `describe_endpoint`, `check_balance`. Paid:
`fetch_data` (any endpoint above), `topup_credits`, plus the named shortcut
`google_search`. Drop the `headers` block to pay per call with x402 instead —
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

- Full agent setup: <https://google.fetcher.sh/skill.md>
- OpenAPI 3.1 contract: <https://google.fetcher.sh/openapi.json>
- Condensed catalog: <https://google.fetcher.sh/llms.txt>
- Payment, credits, and MCP deep dive: [`fetcher` skill](../fetcher/SKILL.md)
- Site: <https://google.fetcher.sh>
