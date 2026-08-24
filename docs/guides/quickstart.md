---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
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

This guide gets you from zero to a confirmed order in under ten minutes. It uses order-api **2.4.0**. If you are on an earlier version, add `size_system` to your requests now — it becomes required for multi-facility accounts in 2.4 and for all accounts in **2.6**.

## Prerequisites

- An API key for your Printf account (`acct_…`)
- A design ID (`dsn_…`) — create one in the dashboard
- A facility ID (`fac-…`) — find yours under **Account → Facilities**

## 1. Place your first order

Send a `POST` to `/v2/orders`. Every field shown below is required.

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

Required fields at a glance:

| Field | Level | Description |
|---|---|---|
| `accountId` | Order | Your account identifier |
| `destination` | Order | Shipping address object |
| `size_system` | Order | `US`, `EU`, or `JP` — applies to all lines unless overridden |
| `designId` | Line | The design to print |
| `garmentSku` | Line | The garment to print on |
| `quantity` | Line | Number of units |
| `size` | Line | Size label (`S`, `M`, `L`, `XL`, …) |
| `size_system` | Line (optional) | Overrides the order-level `size_system` for this line |
| `fit` | Line (optional) | `unisex` (default), `mens`, or `womens` |

## 2. Read the response

A `201` response confirms the order. Each line includes a `resolved_size` object — verify it before sending fulfilment confirmations to recipients.

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

The `X-Printf-Size-System` response header echoes the size system that resolved the order (`US`, `EU`, `JP`, or `mixed` for multi-system orders).

## 3. Handle errors

| Code | HTTP status | What it means | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Your account can route to more than one facility and no `size_system` was provided | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account; bare size label was accepted but is deprecated | Add `size_system` before **2.6** |

## 4. Next steps

- **Bulk orders**: place multiple lines in a single call — see [Bulk orders and templates](bulk-orders.md).
- **Webhooks**: subscribe to `order.fulfilled` to receive `resolved_size` on each line automatically.
- **Size reference**: full size ladders for US, EU, and JP are in [Sizing and fit](sizing.md).

