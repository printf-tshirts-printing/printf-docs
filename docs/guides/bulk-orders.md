---
title: Bulk orders and templates
section: guides
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

# Bulk orders and templates

This guide covers high-volume orders and saved order templates. **If you use saved templates, read the size system section carefully** — templates do not surface in the API response and will not automatically gain a `size_system` value after the 2.4.0 upgrade.

## Bulk order payload

A bulk order follows the same shape as a single order. Supply `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`. Add `size_system` and `fit` to avoid ambiguity errors:

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
      "garmentSku": "tee-classic-black",
      "size": "S",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 300
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 500
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 400
    },
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

Each response line includes `resolved_size` so you can verify the chest measurement before fulfilment begins:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

## Saved order templates

> ⚠️ **Action required for template users.** Saved templates are not visible in the API. You must open each template in the dashboard and add an explicit `size_system`. A template without `size_system` will produce a `size_system_implicit` warning on every order it creates today, and will be **rejected with an error in 2.6.0**.

Templates that route to multiple facilities will already produce `400 size_system_ambiguous` if they omit `size_system`. There is no grace period for multi-facility accounts.

### Auditing your templates

1. Open **Dashboard → Templates**.
2. For each template, check whether `size_system` is set at the order level or on every line.
3. If absent, add `size_system` matching the intended market (see the [Sizing and fit guide](sizing.md) for the system ladder comparison).
4. Save the updated template.

## Rate limits for bulk submissions

Bulk orders count against the same per-account rate limit as single orders. If you are submitting multiple bulk orders in a loop, add the `X-Printf-Size-System` response header check to your retry logic — a `size_system_ambiguous` error will not resolve on retry without a payload change.

## Error and warning codes

| Code | HTTP status | Meaning | Action |
|------|-------------|---------|--------|
| `size_system_ambiguous` | 400 | Multi-facility account omitted `size_system` | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account; system inferred from facility | Add explicit `size_system` before 2.6.0 |

