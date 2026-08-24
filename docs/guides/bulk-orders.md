---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers placing large orders programmatically and managing saved order templates. If you are upgrading from an API version before 2.4.0, read the sizing section below before running any bulk job or reusing an existing template.

## Placing a bulk order

Each line in a bulk order must include `designId`, `garmentSku`, `quantity`, and `size`. As of **2.4.0**, each line must also resolve to an unambiguous size system. Set `size_system` at the order level to cover every line at once:

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
      "fit": "unisex",
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "fit": "unisex",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "fit": "unisex",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

You can override `size_system` per line when a single order spans multiple markets:

```json
{
  "designId": "dsn_7fa91c",
  "size": "XL",
  "size_system": "EU",
  "fit": "womens",
  "quantity": 60,
  "garmentSku": "tee-classic-black"
}
```

## Verifying resolved sizes before fulfillment

The response body includes `resolved_size` on every line. Check this before treating the order as confirmed:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk jobs, iterate `lines[].resolved_size` and verify `system` matches your intent. A mismatch at this stage is recoverable; a mismatch at fulfillment is not.

The `X-Printf-Size-System` response header gives you the order-level effective system in one string, useful for logging.

## Saved order templates and 2.4.0

> ⚠️ **Action required if you use saved templates.**

Templates are evaluated at submission time. A template saved before 2.4.0 that has no `size_system` field will behave as follows after you upgrade:

| Account routing | Result |
|---|---|
| Routes to more than one facility | `400 size_system_ambiguous` — order rejected |
| Routes to exactly one facility | Order accepted with `size_system_implicit` warning |

The `size_system_implicit` warning becomes a hard `400` error in **2.6**.

**To fix a saved template:** open the template and add `size_system` at the top level, at the line level, or both. There is no API endpoint that lists template fields — you must open each template manually and inspect it.

## Error reference for bulk jobs

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` is absent and account can route to multiple facilities. Add `size_system`. |
| `size_system_implicit` | — (warning) | `size_system` is absent but resolved from single facility default. Add `size_system` before 2.6. |

