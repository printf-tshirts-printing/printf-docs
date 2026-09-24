---
title: Quickstart
section: guides
last_reviewed: 2026-09-24
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Quickstart

This guide walks you through placing your first order with the Printf Orders API.

## Before you begin

- An API key provisioned for your account
- Your `accountId` (format: `acct_…`)
- A `designId` for the design you want to print
- A `facilityId` for the fulfilment facility (find yours in the dashboard)

## Place your first order

As of 2.4.0, every order requires `size_system`. The API supports `US`, `EU`, and `JP`.

```json
POST /v2/orders
Authorization: Bearer <your-api-key>

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

### What comes back

HTTP 201. Each line in the response includes `resolved_size`, confirming the size ladder that was applied:

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

The `X-Printf-Size-System` response header also reflects the system applied to the order.

## Required fields

| Field | Level | Notes |
|-------|-------|-------|
| `accountId` | Order | Your account identifier. |
| `size_system` | Order or line | `US`, `EU`, or `JP`. Required from 2.4.0; line-level overrides order-level. |
| `facilityId` | Order | The fulfilment facility. |
| `destination` | Order | Full address object; see field list above. |
| `designId` | Line | The design to print. |
| `garmentSku` | Line | The specific garment. |
| `quantity` | Line | Units to produce. |

## Error codes you will hit early

| Code | HTTP status | Fix |
|------|-------------|-----|
| `size_system_ambiguous` | 400 | Add `size_system` to your request. Your account routes to more than one facility and no default can be inferred. |
| `size_system_implicit` | — (warning) | `size_system` is absent but was inferred from your account or facility default. Add it explicitly. Becomes an error in 2.6. |

## Next steps

- [Sizing and fit](/guides/sizing) — size system precedence, `fit`, and `resolved_size` in detail
- [Bulk orders and templates](/guides/bulk-orders) — submit many orders at once and manage saved templates
