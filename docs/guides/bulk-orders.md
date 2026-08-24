---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-rb
  - printf-go
---

# Bulk orders and templates

Bulk orders follow the same request shape as single orders. This guide covers high-volume patterns and template management, updated for **order-api 2.4.0** size disambiguation.

## Required fields

Every order, bulk or single, requires:

| Field | Level | Notes |
|-------|-------|-------|
| `accountId` | Order root | |
| `destination` | Order root | Full address object |
| `designId` | Line | |
| `garmentSku` | Line | Full SKU string — do not abbreviate |
| `quantity` | Line | |

From 2.4.0, you should also always include:

| Field | Level | Notes |
|-------|-------|-------|
| `size_system` | Order root or line | `US`, `EU`, or `JP`. Omitting triggers `size_system_implicit` warning; becomes error in 2.6. |
| `fit` | Line | `unisex`, `mens`, or `womens`. |

## Bulk request example

Pass multiple lines in the `lines` array. Each line resolves `size_system` independently if set; otherwise it inherits from the order root.

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
      "size": "S",
      "size_system": "US",
      "fit": "womens",
      "quantity": 100,
      "garmentSku": "tee-classic-white"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-slim-navy"
    }
  ]
}
```

Note that the third line overrides `size_system` to `JP`. A JP `M` resolves to a different chest measurement than a US `M` — set this deliberately.

## Saved order templates

Templates are stored server-side and are **not visible in the API**. This makes them easy to forget during upgrades.

**Action required for 2.4.0:** Any template that lacks `size_system` will produce a `size_system_implicit` warning on every fulfillment triggered from it. This warning becomes `400 size_system_ambiguous` in 2.6 for multi-facility accounts, and a hard error for all accounts.

To audit templates:
1. Contact Printf support or use the dashboard template manager.
2. Add `size_system` at the order level of every template.
3. Add `fit` to each line where the cut matters.
4. Re-save the template.

There is no API endpoint for listing saved templates today. Do not assume a template is correct because a past order succeeded.

## `resolved_size` in bulk responses

Each line in the response includes `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

The `X-Printf-Size-System` header reflects the system applied at the order level (the root-level `size_system` or the resolved fallback).

## Error: `size_system_ambiguous`

Multi-facility accounts that omit `size_system` entirely receive `400 size_system_ambiguous`. The fix is to add `size_system` at the order root or on every line. This is not a warning — the order is not created.

