---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: dx
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

This guide takes you from zero to a submitted order in under ten minutes.
Examples target **API 2.4.0**. If you are on an earlier version, see the migration note for
`size_system` before proceeding.

## Prerequisites

- An API key (get one at **Dashboard → Settings → API keys**)
- Your `accountId` (shown on the account overview page)
- A published design — you need its `designId`
- A facility — you need its `facilityId`

## Submit your first order

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Required fields on every request:

| Field | Where | Notes |
|---|---|---|
| `accountId` | Order | Your account identifier |
| `destination` | Order | Full shipping address object |
| `size_system` | Order and/or line | `US`, `EU`, or `JP` — **required as of 2.4.0** |
| `designId` | Line | Published design identifier |
| `garmentSku` | Line | Garment product SKU |
| `quantity` | Line | Units to produce |

## Read the response

A `201 Created` response includes a `resolved_size` object on each line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Check `resolved_size.system` to confirm which size ladder was applied. The response header
`X-Printf-Size-System` reports the same value at the order level.

## Common errors

| Code | Meaning | Fix |
|---|---|---|
| `400 size_system_ambiguous` | Multi-facility account, no `size_system` set | Add `size_system` to the order or every line |
| `size_system_implicit` (warning) | Single-facility account, no `size_system` set | Add `size_system` — becomes an error in 2.6 |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — size system reference, ladders, and fit values
- [Bulk orders and templates](docs/guides/bulk-orders.md) — submit many orders at once, update saved templates

