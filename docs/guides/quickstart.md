---
title: Quickstart
section: Guides
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

This guide walks you through placing your first order with the Printf API. It targets **order-api 2.4.0**. If you are on an earlier version, see the migration note at the bottom before proceeding.

## Prerequisites

- A Printf account with an `accountId`.
- An API key with `orders:write` scope.
- A design ID (`designId`) from the Printf dashboard.

## Place your first order

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

Required fields: `accountId`, `destination`, and in each line: `designId`, `garmentSku`, `quantity`.

`size_system` (`US`, `EU`, or `JP`) is required from **2.6** and strongly recommended now. Omitting it on an account routable to multiple facilities returns `400 size_system_ambiguous` immediately.

## Read the response

A `201 Created` response includes a `resolved_size` on each line:

```json
{
  "orderId": "ord_abc123",
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

Check the `X-Printf-Size-System` response header to confirm which size system was applied to the order.

## Common errors

| Error code | Meaning | Fix |
|------------|---------|-----|
| `size_system_ambiguous` | Account routes to multiple facilities; `size_system` was not set. | Add `size_system` at the order root or on each line. |
| `size_system_implicit` | `size_system` was inferred from a single facility. | Add `size_system` explicitly. Becomes an error in 2.6. |

## Migrating from before 2.4.0

If your existing integration omits `size_system` and `fit`, see the [migration note in the sizing guide](sizing.md) for a before/after diff and affected warning codes.

