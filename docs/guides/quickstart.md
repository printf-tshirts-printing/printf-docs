---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-rb
  - printf-go
---

# Quickstart

This guide takes you from zero to a submitted order in under ten minutes. It uses order-api **2.4.0**.

## Prerequisites

- A Printf account — `accountId` is shown on your dashboard.
- An API key from **Settings → API Keys**.
- At least one saved design — `designId` is shown on the design detail page.
- A garment SKU from the [product catalog](https://app.getport.io).

## Your first order

Send a `POST` to `/v2/orders`. All five top-level required fields must be present: `accountId`, `destination`, and `lines` (each line requires `designId`, `garmentSku`, and `quantity`). As of 2.4.0, include `size_system` at both order and line level.

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

## What comes back

A `200` response includes the booked order. Each line now contains a `resolved_size` object confirming the exact measurements the fulfillment system will use:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Also check the `X-Printf-Size-System` response header — it confirms the system applied to your order.

## Common first-order errors

| Code | What it means | Fix |
|---|---|---|
| `size_system_ambiguous` | `size_system` missing, account routes to multiple facilities | Add `size_system` (`US`, `EU`, or `JP`) to the order and each line |
| `size_system_implicit` | `size_system` missing, single-facility account (warning for now) | Add `size_system` before it becomes a hard error in 2.6 |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full size ladder reference and resolution fallback chain.
- [Bulk orders and templates](docs/guides/bulk-orders.md) — sending batches and updating saved templates.
- [Webhooks](docs/guides/webhooks.md) — `resolved_size` is now included in all webhook payloads.

