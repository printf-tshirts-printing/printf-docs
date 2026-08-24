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
  - printf-go
  - printf-rb
---

# Quickstart

This guide takes you from zero to a confirmed order in fewer than ten minutes. It uses API version **2.4.0**. If you are on an earlier version, the payload shape is different — see the [sizing and fit guide](docs/guides/sizing.md) for the migration.

## Prerequisites

- An API key for your account
- Your `accountId` (format: `acct_…`)
- A design uploaded to your account (`designId`, format: `dsn_…`)
- A `garmentSku` from the product catalog

## 1. Place your first order

Send a `POST` to `/v2/orders`. All five of `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required. `size_system` is required as of 2.4.0.

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

## 2. Read the response

A `201 Created` response confirms the order. Each line includes `resolved_size` — verify it before proceeding:

```json
{
  "orderId": "ord_…",
  "status": "confirmed",
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

Also check the `X-Printf-Size-System` response header — it reflects the effective size system used for the order.

## 3. What to do if the request is rejected

| Error code | Fix |
|---|---|
| `size_system_ambiguous` | Add `size_system` (`US`, `EU`, or `JP`) to the order body or to each line |
| `size_system_implicit` | Warning only — your order was accepted, but add `size_system` before upgrading to 2.6 |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full size-system reference, fit values, chest measurements
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line orders and saved template migration

