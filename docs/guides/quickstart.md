---
title: Quickstart
section: guides
last_reviewed: 2026-08-23
owner: platform-docs
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

This guide gets you from zero to a confirmed order in five minutes. It uses **order-api 2.4.0**. If you are on an older version, see the migration note for `size_system_ambiguous` before continuing.

## Prerequisites

- An account ID (`acct_…`)
- An API key
- At least one design ID (`dsn_…`) and a garment SKU

## Step 1 — Authenticate

All requests require `Authorization: Bearer <your-api-key>` in the header.

## Step 2 — Place your first order

`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every request. As of 2.4.0, you must also supply `size_system` on the order or on each line.

```json
POST /v2/orders
Authorization: Bearer <your-api-key>
Content-Type: application/json

{
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
}
```

## Step 3 — Inspect the response

A `201` response includes `resolved_size` on every line. Check it before treating the order as confirmed.

```json
{
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "resolved_size": {
        "label": "XL",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 112
      }
    }
  ]
}
```

The response header `X-Printf-Size-System` tells you which size system was applied to the request.

## Step 4 — Handle errors

| Code | HTTP | What to do |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to the order or to each line. |
| `size_system_implicit` | — (warning) | Add `size_system` now. It becomes a hard error in 2.6. |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full ladder tables, `fit` values, and `resolved_size` semantics
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line and multi-order payloads, saved template update checklist

