---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Quickstart

This guide walks you through placing your first order against the Printf API. It reflects **API 2.4.0**.

## Prerequisites

- An active Printf account (`accountId`)
- An API key (set as `Authorization: Bearer <key>`)
- A design uploaded via the dashboard or the Designs API (`designId`)

## Place your first order

Send a `POST` to `/v2/orders`. Every request requires `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`.

As of 2.4.0, you must also supply `size_system` — either at the order level, per line, or both.

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

## Read the response

A `201 Created` response contains the order object. Each line now includes a `resolved_size` object — check it to confirm the size system and physical dimensions used for fulfillment:

```json
{
  "orderId": "ord_...",
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

The `X-Printf-Size-System` response header contains the effective size system applied to the order.

## Required fields at a glance

| Field | Level | Required |
|---|---|---|
| `accountId` | Order | ✅ Always |
| `destination` | Order | ✅ Always |
| `size_system` | Order and/or Line | ✅ As of 2.4.0 (warning in 2.4, error in 2.6 if omitted) |
| `designId` | Line | ✅ Always |
| `garmentSku` | Line | ✅ Always |
| `quantity` | Line | ✅ Always |
| `fit` | Line | Optional (defaults to garment catalogue default) |

## Common errors

| Code | What it means | Fix |
|---|---|---|
| `size_system_ambiguous` | Your account routes to multiple facilities and `size_system` cannot be inferred | Add `size_system` explicitly to the order or each line |
| `size_system_implicit` | `size_system` was inferred; will become an error in 2.6 | Add `size_system` explicitly |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full size system reference, `fit` values, and `resolved_size` semantics
- [Bulk orders and templates](docs/guides/bulk-orders.md) — sending multiple orders and updating saved templates
