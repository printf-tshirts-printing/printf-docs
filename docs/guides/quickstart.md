---
title: Quickstart
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Quickstart

This guide gets you from zero to a confirmed order in under ten minutes. It reflects **API 2.4.0** — if you are on an older version, the `size_system` and `fit` fields are new; see [Sizing and fit](./sizing.md) for background.

## Prerequisites

- An API key (Settings → API keys in the dashboard)
- An account ID (`acct_…`) and at least one design ID (`dsn_…`)
- A `garmentSku` — find these in the product catalog under your account

## Your first order

```bash
curl -X POST https://api.printf.dev/v2/orders \
  -H "Authorization: Bearer $PRINTF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "acct_stackfest",
    "size_system": "US",
    "facilityId": "fac-atx",
    "destination": {
      "name": "StackFest Ops",
      "line1": "410 Congress Ave",
      "city": "Austin",
      "region": "TX",
      "postalCode": "78701",
      "countryCode": "US"
    },
    "lines": [
      {
        "designId": "dsn_7fa91c",
        "size": "XL",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }'
```

A successful response is `201 Created`. Every line in the response body includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Confirm that `chest_cm` matches your expectation. If it does not, the `size_system` in your request resolved differently than you intended — check the response header `X-Printf-Size-System` to see what the API applied at the order level.

## Required fields

Every request to `POST /v2/orders` must include:

| Field | Type | Notes |
|---|---|---|
| `accountId` | string | Your account identifier |
| `destination` | object | Full address including `countryCode` |
| `lines[].designId` | string | Design to print |
| `lines[].garmentSku` | string | Exact SKU from the product catalog |
| `lines[].quantity` | integer | Units |

`size_system` is not required but you should always set it. Omitting it on an account that routes to multiple facilities returns `400 size_system_ambiguous`.

## Common errors

| Code | HTTP | Cause | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` resolved | Add `size_system` to the order or the line |
| `size_system_implicit` | — | Single-facility account; resolved via facility default | Warning now, error in 2.6 — add `size_system` |
| `invalid_garment_sku` | 422 | `garmentSku` not in catalog | Check spelling and catalog availability |
| `design_not_found` | 404 | `designId` not found on this account | Verify the design exists and belongs to `accountId` |

## Next steps

- [Sizing and fit](./sizing.md) — size systems, ladders, and `resolved_size` in depth
- [Bulk orders and templates](./bulk-orders.md) — large orders and saved template migration
- [Webhooks](./webhooks.md) — `resolved_size` now appears in webhook payloads too

