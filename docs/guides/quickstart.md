---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
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

This guide gets you from zero to a submitted order. It targets **order-api 2.4**. If you are on an older version, see the migration note below before continuing.

## Prerequisites

- A Printf account with at least one approved design
- Your `accountId`, a `designId`, and a `garmentSku`
- An API key in the `Authorization: Bearer <token>` header

## Your first order

Submit a `POST /v2/orders` request. Every field shown below is required.

```json
POST /v2/orders
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

A successful `201` response includes `resolved_size` on each line and the `X-Printf-Size-System` header.

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

Confirm `resolved_size.system` matches what you sent. If it does not, the API resolved the system from your account or facility default — add `size_system` explicitly to avoid surprises.

## Required fields

| Field | Level | Description |
|---|---|---|
| `accountId` | order | Your Printf account identifier |
| `destination` | order | Full shipping address object |
| `size_system` | order or line | `US`, `EU`, or `JP` |
| `designId` | line | The approved design to print |
| `garmentSku` | line | The garment to print on |
| `size` | line | Size label, e.g. `S`, `M`, `L`, `XL` |
| `quantity` | line | Units to produce |
| `fit` | line | Garment fit, e.g. `unisex` |

`size_system` is required at the order level, the line level, or both. Omitting it on a multi-facility account returns `400 size_system_ambiguous`.

## Size systems

A JP `XL` is 97 cm chest. A US `XL` is 112 cm. Always specify the system that matches your recipients' market — wrong system = wrong garment delivered.

| System | Markets |
|---|---|
| `US` | United States, Canada |
| `EU` | Europe |
| `JP` | Japan |

## Next steps

- [Sizing and fit](sizing.md) — full size ladders, fit types, and mixed-system orders
- [Bulk orders and templates](bulk-orders.md) — multi-line orders, saved templates, and the 2.4 template audit checklist
- [Webhooks](../sdks/webhooks.md) — `resolved_size` now appears on webhook payloads too
