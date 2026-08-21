---
title: Quickstart
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: ["POST /v2/orders"]
covers_sdks: ["printf-js","printf-py","printf-java","printf-go","printf-rb"]
---

# Quickstart

This guide walks you through placing your first order with the Printf API.

## Prerequisites

- A Printf account (`accountId`)
- An API key
- At least one design uploaded (`designId`)

## Place your first order

```json
POST /v2/orders
Authorization: Bearer <api_key>

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

> **`size_system` is required from 2.6 onward.** It is strongly recommended now. Omitting it on a multi-facility account returns `400 size_system_ambiguous`. Omitting it on a single-facility account returns a `size_system_implicit` warning that becomes an error in 2.6.

## Read `resolved_size` from the response

Every line in the response carries a `resolved_size` object confirming exactly what was interpreted:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header also tells you which system resolved at the order level.

## Next steps

- [Sizing and fit](sizing.md) — full fallback chain, per-line overrides, size ladder tables
- [Bulk orders and templates](bulk-orders.md) — multi-line orders and saved template audit
- [Webhooks](webhooks.md) — `resolved_size` appears in webhook payloads too

