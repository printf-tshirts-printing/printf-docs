---
title: Quickstart
section: guides
last_reviewed: 2026-08-22
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

This guide gets you from zero to a submitted order in under five minutes. It uses order-api **2.4.0**. If you are on an earlier version, the `size_system` and `fit` fields below are not recognised — upgrade first.

## Prerequisites

- An API key scoped to your account (`accountId`)
- A design ID (`designId`) from the Printf dashboard
- A garment SKU (`garmentSku`) from the product catalogue

## Your first order

Submit a `POST /v2/orders`. All five of `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required. `size_system` is required from **2.6** onward and strongly recommended now.

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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

## Reading the response

A `201 Created` response includes a `resolved_size` object on each line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Confirm `chest_cm` matches the garment you expect. The response header `X-Printf-Size-System` echoes the system that was resolved for the order.

## Common errors

| Code | What it means | Fix |
|---|---|---|
| `size_system_ambiguous` | Your account routes to multiple facilities and no `size_system` was provided | Add `size_system` at the order or line level |
| `size_system_implicit` | `size_system` was inferred from your single facility's default (warning; error in 2.6) | Add `size_system` explicitly |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full size system reference, fit options, and ladder tables
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line orders and saved template migration

