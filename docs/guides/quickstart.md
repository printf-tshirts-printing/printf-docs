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

This guide gets you from zero to a placed order in five minutes. It uses the 2.4.0 API. If you are on an older integration, read the [sizing and fit guide](sizing.md) before proceeding — `size` now requires an explicit `size_system`.

## Prerequisites

- A Printf account (`accountId`) and API key.
- A design uploaded via the design API (`designId`).
- A garment SKU from the garment catalogue (`garmentSku`).

## Step 1 — Place your first order

```http
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

Required fields on every request:

| Field | Where | Notes |
|---|---|---|
| `accountId` | Order body | Your Printf account identifier |
| `destination` | Order body | Full shipping address object |
| `designId` | Each line | The design to print |
| `garmentSku` | Each line | The exact garment identifier |
| `quantity` | Each line | Units to produce |

New in 2.4.0:

| Field | Where | Notes |
|---|---|---|
| `size_system` | Order body and/or line | `US`, `EU`, or `JP`. Set it. |
| `fit` | Each line | `unisex` works for all garments |

## Step 2 — Read the response

A successful `201` response includes `resolved_size` on every line:

```json
{
  "orderId": "ord_abc123",
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

Check `resolved_size.chest_cm` against your expectations. If the number surprises you, your `size_system` or `garmentSku` is wrong — cancel and resubmit rather than waiting for fulfilment.

The response also includes the header:

```
X-Printf-Size-System: US
```

## Step 3 — Handle errors

The two errors you are most likely to hit on a fresh integration:

| Code | HTTP | Fix |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Add `size_system` now; becomes `400` in 2.6 |

## Next steps

- [Sizing and fit](sizing.md) — full size system reference and ladder tables
- [Bulk orders and templates](bulk-orders.md) — high-volume workflows and template migration

