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
  - printf-go
  - printf-rb
---

# Quickstart

This guide gets you to a confirmed order in five minutes.

> **Running 2.4.0 or later?** The order body below includes the required `size_system` and `fit` fields. If you are on an older version, remove them — but plan to add them before 2.6.

## Before you start

You need:

- An API key (from the dashboard under **Settings → API keys**)
- An `accountId` — shown on the account overview page
- A `facilityId` — shown on the facility detail page, or ask your onboarding contact
- A `designId` — created via the designs API or uploaded in the dashboard

## Send your first order

```bash
curl -X POST https://api.printf.dev/v2/orders \
  -H "Authorization: Bearer $PRINTF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
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
  }'
```

## Read the response

A successful response has HTTP 201. Each line includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Always check `chest_cm`. It is the physical measurement that will be produced. A JP XL and a US XL share the same label but differ by 15 cm.

The response also includes the `X-Printf-Size-System` header, echoing the size system applied to the order.

## What can go wrong

| Code | What it means | Fix |
|---|---|---|
| `size_system_ambiguous` | Your account routes to multiple facilities and no size system was declared | Add `size_system` to the order or each line |
| `size_system_implicit` | Size system was inferred from your single facility's default — will become an error in 2.6 | Add `size_system` explicitly |

## Next steps

- [Sizing and fit](sizing.md) — full size-system reference and ladder tables
- [Bulk orders and templates](bulk-orders.md) — submit many lines at once; update saved templates for 2.4.0

