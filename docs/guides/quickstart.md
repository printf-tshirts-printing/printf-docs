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

This guide gets you from zero to a confirmed order in under five minutes. It targets **order-api 2.4.0**. If you are on an older integration, see the [migration note](../MIGRATION-2.4.0.md) before continuing — the size fields are now required.

## Prerequisites

- A Printf account with at least one approved design
- Your `accountId` (find it in the Printf dashboard under Settings → Account)
- An API key with `orders:write` scope

## Step 1 — Create your first order

Send a `POST` to `/v2/orders`. Every request requires `accountId`, `destination`, and at least one line. Each line requires `designId`, `garmentSku`, and `quantity`. As of 2.4.0, `size_system` is also required (either on the order or per line).

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

## Step 2 — Read the response

A `201 Created` response means the order is accepted. Check the `resolved_size` object on each line to confirm the API interpreted the size exactly as you intended:

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

Also inspect the `X-Printf-Size-System` response header — it confirms the size system applied to the order as a whole.

## Step 3 — Handle errors

| Code | HTTP status | What to do |
|---|---|---|
| `size_system_ambiguous` | 400 | Add an explicit `size_system` to your request at the order level or on every line. |
| `size_system_implicit` | — (warning in response body) | Your request worked, but `size_system` was inferred rather than explicit. Add it to your payload before 2.6, when this warning becomes an error. |

## Next steps

- [Sizing and fit](sizing.md) — full resolution chain, fit options, and chest measurements by system
- [Bulk orders and templates](bulk-orders.md) — high-volume submission and updating saved templates

