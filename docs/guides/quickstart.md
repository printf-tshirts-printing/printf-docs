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

Place your first order in under five minutes.

## Prerequisites

- An API key from the [Printf dashboard](https://app.printf.dev/settings/api-keys).
- Your `accountId` (shown on the account overview page).
- At least one saved design (`designId`) and a garment SKU (`garmentSku`).

## Your first order

Send a `POST` to `/v2/orders`. The fields `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required. As of **2.4**, you should also include `size_system` — bare size labels without a resolvable system are rejected for multi-facility accounts and will be rejected for all accounts in **2.6**.

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

## Reading the response

A `201` response confirms the order was accepted. Each line includes `resolved_size` so you can verify what was actually recorded:

```json
{
  "orderId": "ord_...",
  "lines": [
    {
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

The response also includes the header:

```
X-Printf-Size-System: US
```

## Size systems

| `size_system` | XS chest (cm) | S | M | L | XL |
|---|---|---|---|---|---|
| `US` | 84 | 91 | 99 | 107 | 112 |
| `EU` | 82 | 88 | 96 | 102 | 107 |
| `JP` | 78 | 84 | 90 | 94 | 97 |

Ladders differ materially at every size. Set `size_system` explicitly.

## If your request is rejected

| Error code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | Your account can route to more than one facility and no size system was provided. | Add `size_system` to your order or to each line. |
| `size_system_implicit` | Size system resolved from account or facility default (warning today, error in 2.6). | Add `size_system` explicitly. |

## Next steps

- [Sizing and fit](./sizing.md) — full ladder reference, `fit` values, and the resolution chain.
- [Bulk orders and templates](./bulk-orders.md) — submitting multiple line items and updating saved templates.
- [Webhooks](./webhooks.md) — `resolved_size` is also present on webhook payloads.
