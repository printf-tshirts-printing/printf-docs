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

This guide gets you from zero to a confirmed order in under ten minutes.

## Prerequisites

- An account ID (`accountId`) from your Printf dashboard
- An API key in the `Authorization` header (`Bearer <key>`)
- A design ID (`designId`) for the artwork to print

## Send your first order

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

`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required
on every request. `size_system` is required as of **2.4.0** for all accounts that
route to more than one facility — and strongly recommended for all accounts.

## What you get back

```json
{
  "orderId": "ord_...",
  "status": "accepted",
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

The `resolved_size` object on each line confirms the exact measurement selected.
The `X-Printf-Size-System` response header shows the system applied to the order.

## Size systems

Printf supports three size systems. Set the one that matches your end-wearers:

| Value | Region | US XL chest |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 109 cm |
| `JP` | Japan | 97 cm |

Set `size_system` at order level as a default, or per line to mix systems in one
order.

## Common errors

| Code | HTTP status | Fix |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to the order or each line |
| `size_system_implicit` | warning | Add `size_system` before 2.6 when it becomes an error |

## Next steps

- [Sizing and fit](sizing.md) — full size ladder tables and `fit` options
- [Bulk orders and templates](bulk-orders.md) — multi-line orders and saved templates

