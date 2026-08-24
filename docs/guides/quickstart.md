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

This guide walks you through placing your first order against the Printf API. It reflects **API version 2.4.0**. If you are on an older version, the `size_system` and `fit` fields below are new — add them now to avoid errors when you upgrade.

## Prerequisites

- An API key for your account. Generate one from the Printf dashboard under **Settings → API keys**.
- Your `accountId` (shown on the dashboard home screen as **Account ID**).
- The `facilityId` for the fulfillment location you want to target. If you have only one facility linked to your account, you can omit `facilityId` and Printf will route automatically.

## Place your first order

```http
POST /v2/orders
Authorization: Bearer <your-api-key>
Content-Type: application/json
```

```json
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

**Required fields on every request:** `accountId`, `destination`, `designId` (per line), `garmentSku` (per line), `quantity` (per line).

**New in 2.4.0:** `size_system` and `fit`. You can set `size_system` at the order root (applies to all lines) and override it per line. `fit` is per-line only and defaults to `unisex` if omitted.

## Read the response

A successful request returns `201 Created`. Each line in the response body includes a `resolved_size` object:

```json
{
  "orderId": "ord_91abc4",
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

Always check `resolved_size.chest_cm` against your design spec before treating an order as final. A JP `XL` is 97 cm; a US `XL` is 112 cm. They are different garments.

The `X-Printf-Size-System` response header contains the system applied at the order level. Save it in your logs.

## Common errors

| Code | HTTP status | Fix |
|------|-------------|----|
| `size_system_ambiguous` | 400 | Add `size_system` to the order root or to each line. Affects accounts that can route to more than one facility. |
| `size_system_implicit` | — (warning) | Your request succeeded, but `size_system` was inferred from your facility's default. Add it explicitly — this becomes a hard error in **2.6**. |

## Next steps

- [Sizing and fit](sizing.md) — full size ladder reference, per-line overrides, and the `fit` field.
- [Bulk orders and templates](bulk-orders.md) — adding multiple lines, and how to update saved templates for 2.4.0.

