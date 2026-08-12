---
name: x-api-endpoints
description: Full parameter reference for every X (Twitter) endpoint on twitter.fetcher.sh — path params, query params, required flags, and enum values, straight from the live OpenAPI contract.
---

# X — full endpoint reference

Base URL: `https://twitter.fetcher.sh` (same host X used before the rebrand —
fetcher.sh never moved it). All 15 endpoints are `GET`, authenticated with
either `Authorization: Bearer bby_live_...` (credits) or an x402 payment —
see [`SKILL.md`](../SKILL.md) for the payment flow. Path parameters are
written as `{param}`; substitute the real value. `?` marks an optional query
param.

This file lists every parameter that exists. It does not describe response
fields — the response shape is generated at request time from the live
handlers, so the authoritative machine-readable version is always
[`/openapi.json`](https://twitter.fetcher.sh/openapi.json). Treat any field
name you haven't seen in an actual response as unverified.

## Search

| Endpoint | Params |
| --- | --- |
| `/api/search` | `query` (required) · `sort`? (`Latest`, `Top`) · `cursor`? |
| `/api/search/users` | `query` (required) · `cursor`? |

`query` on `/api/search` is passed straight to X's own search parser — every
operator X supports (`from:`, `to:`, `since:`, `until:`, `min_faves:`,
`min_retweets:`, `filter:`, `-filter:`) works unmodified.

## Profiles

| Endpoint | Params |
| --- | --- |
| `/api/handle/{handle}` | none |
| `/api/handle/{handle}/about` | none |
| `/api/user/{id}` | none |

`{handle}` takes the `@username` without the `@`. `{id}` is the numeric user
ID returned by the handle lookup — endpoints below that take `{id}` need this
value, not the handle.

## Timelines and relationships

| Endpoint | Params |
| --- | --- |
| `/api/user/{id}/tweets` | `cursor`? |
| `/api/user/{id}/replies` | `cursor`? |
| `/api/user/{id}/followers` | `cursor`? |
| `/api/user/{id}/followings` | `cursor`? |

All four paginate with an opaque `cursor` — pass back exactly what the
previous response returned, never construct or decode it.

## Posts

| Endpoint | Params |
| --- | --- |
| `/api/tweet/{id}` | none |
| `/api/tweet/{id}/replies` | `cursor`? |
| `/api/tweet/{id}/retweeters` | `cursor`? |

`{id}` is the post's numeric status ID (the number in an X URL). Treat it as
a string — it can exceed `Number.MAX_SAFE_INTEGER`, so parsing it as a JS
number silently corrupts it. The path segment is still `tweet` — the API
predates the "post" rename and fetcher.sh mirrors the live route as-is.

## Lists and trends

| Endpoint | Params |
| --- | --- |
| `/api/list/{id}/members` | `cursor`? |
| `/api/list/{id}/tweets` | `cursor`? |
| `/api/trends` | `country` (required) |

`{id}` for lists is the numeric list ID from an `x.com/i/lists/{id}` URL.
`country` on `/api/trends` takes a country name (e.g. `United States`), not
an ISO code.

## Pricing

Every endpoint above is $0.005/call except `/api/tweet/{id}`, which is
$0.002/call — the cheapest way to resolve a single post ID you already have.

## See also

- [`scenarios.md`](scenarios.md) — one worked `curl` per endpoint
- [`faq.md`](faq.md) — task-oriented "how do I..." answers
- [`comparison.md`](comparison.md) — fetcher.sh vs. the official X API vs. a browser scraper
