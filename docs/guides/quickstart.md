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

Create your first order in under five minutes.

## Prerequisites

- An API key (get one at **Dashboard → Settings → API keys**).
- Your `accountId` (shown on the same page).
- At least one design uploaded — you'll need its `designId`.
- A `garmentSku` from the product catalogue.

## Your first order

```bash
curl -X POST https://api.printf.dev/v2/orders \
  -H "Authorization: Bearer <your-api-key>" \
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
        "garmentSku": "tee-classic-black",
        "size": "XL",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 1
      }
    ]
  }'
```

> **Tip:** `size_system` is accepted at both the order level (as a default for all lines) and per line (overrides the order default for that line only). Always supply it — omitting it produces a warning today and will be an error in a future release.

## What you get back

```json
{
  "orderId": "ord_...",
  "status": "accepted",
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 1,
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

The `resolved_size` object confirms the size system and chest measurement that will be cut. Check it before relying on the order.

The `X-Printf-Size-System: US` response header echoes the resolved system for the whole order.

## Required fields

| Field | Where | Notes |
|-------|-------|-------|
| `accountId` | order root | Your account identifier |
| `destination` | order root | Full address object |
| `designId` | each line | The design to print |
| `garmentSku` | each line | Product SKU from catalogue |
| `quantity` | each line | Units to produce |
| `size_system` | order root or each line | `US`, `EU`, or `JP` — required from 2.6.0, warning now |
| `fit` | each line | `unisex` (default), `mens`, or `womens` |

## Next steps

- [Sizing and fit](sizing.md) — full size ladder comparison and per-line override examples.
- [Bulk orders and templates](bulk-orders.md) — high-volume submissions and saved template migration.
- [Webhooks](../guides/webhooks.md) — receive `resolved_size` in fulfilment events.

