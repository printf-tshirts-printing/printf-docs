---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-22
owner: platform-docs
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

This guide covers submitting large orders and using saved order templates. If you use templates, read the [size system migration note](#size-system-migration-for-templates) before doing anything else — templates store line items and are not automatically updated by a deploy.

## Submitting a bulk order

There is no separate bulk endpoint. Submit a single `POST /v2/orders` with multiple lines. Each line is fulfilled independently; a single line failure does not cancel the order.

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
      "fit": "unisex",
      "quantity": 80
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "M",
      "fit": "unisex",
      "quantity": 120
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "L",
      "fit": "unisex",
      "quantity": 150
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

`size_system` set at the order level applies to every line that does not declare its own. You should always set it at the order level for bulk orders to avoid ambiguity.

## Verifying resolution before committing

For large runs, check `resolved_size.chest_cm` on each line in the response before treating the order as confirmed. The response shape per line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

If `chest_cm` does not match your spec, cancel before fulfillment begins.

## Size system migration for templates

> **Action required if you use saved order templates.**

Saved templates store line items verbatim. A template created before order-api 2.4.0 has no `size_system` field at the order or line level. When that template is submitted:

- **Multi-facility accounts** receive `400 size_system_ambiguous` immediately. The order is rejected.
- **Single-facility accounts** receive a `size_system_implicit` warning and the order succeeds today, but this becomes a `400` error in **order-api 2.6**.

### What to do

1. Retrieve every saved template from your system.
2. Add `size_system` to the order-level fields (recommended: set it once at the top level).
3. Optionally add `size_system` and `fit` per line for precision.
4. Save the updated template.

**Before (broken in 2.4 for multi-facility, warning for single-facility):**

```json
{
  "accountId": "acct_stackfest",
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
      "quantity": 250
    }
  ]
}
```

**After (correct for 2.4+):**

```json
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

## Error reference

| Code | HTTP | Meaning | Remedy |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` at any level | Add `size_system` to the order or each line |
| `size_system_implicit` | warning (400 in 2.6) | Single-facility account, `size_system` resolved from facility default | Add `size_system` before 2.6 |

