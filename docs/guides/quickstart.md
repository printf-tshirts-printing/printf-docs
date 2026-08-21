---
title: Quickstart
section: Guides
last_reviewed: 2026-08-21
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

This guide walks you through placing your first order with the Printf API. The examples use **order-api 2.4.0**. If you are on an earlier version, upgrade before continuing — bare size labels without an explicit `size_system` are rejected for multi-facility accounts from 2.4.0 onward, and become a hard error for all accounts in 2.6.

## 1. Get your credentials

All requests require an `Authorization: Bearer <token>` header. Generate a token in the [dashboard](https://app.printf.dev/settings/tokens).

## 2. Place an order

The minimum required fields on every request are `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`.

From 2.4.0, you must also include `size_system` — either at the order level or per line.

```json
POST /v2/orders
Authorization: Bearer <token>
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

## 3. Read the response

A successful `201` response echoes your order with each line expanded to include `resolved_size`:

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

Verify `resolved_size.chest_cm` before storing the order ID. If the chest measurement does not match your expectation, the `size_system` on the line is set to the wrong value.

## 4. Handle errors

| Code | Meaning | Fix |
|------|---------|-----|
| `400 size_system_ambiguous` | No `size_system` set, account routes to multiple facilities | Add `size_system` to the request |
| `size_system_implicit` warning | No `size_system` set, single-facility account — order accepted but this becomes a `400` in 2.6 | Add `size_system` to the request |

See [Sizing and fit](docs/guides/sizing.md) for the full resolution ladder and ladder measurements.

