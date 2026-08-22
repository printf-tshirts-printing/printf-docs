---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-22
owner: platform-docs
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

This guide covers sending large multi-line orders and managing saved order
templates. As of **2.4.0**, `size_system` and `fit` are required fields on every
line — including lines stored inside saved templates.

## ⚠️ Saved templates and size_system

Saved order templates are not visible through the API. If you have templates
created before 2.4.0, they do not include `size_system` or `fit`. Orders
submitted from those templates will:

- Fail with `400 size_system_ambiguous` if your account routes to more than one facility.
- Succeed with a `size_system_implicit` warning if your account has a single facility — but
  this warning becomes a hard error in **2.6**.

**Action required:** open each saved template in the dashboard, add `size_system`
and `fit` to every line, and save it before upgrading.

## Sending a bulk order

A bulk order is a single `POST /v2/orders` with multiple lines. Each line must
include `designId`, `garmentSku`, `quantity`, `size`, `size_system`, and `fit`.

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
      "fit": "unisex",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 200,
      "garmentSku": "tee-classic-black"
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
      "size": "2XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

You can set `size_system` once at order level and omit it on individual lines
when every line shares the same system. Per-line `size_system` overrides the
order-level value.

## Reading resolved sizes in bulk responses

The response body includes `resolved_size` on every line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

This lets you verify that each garment resolved to the measurement you expected
before the order is committed to production.

## Template checklist for 2.4.0

| Check | Detail |
|---|---|
| Add `size_system` to every line in saved templates | `US`, `EU`, or `JP` |
| Add `fit` to every line in saved templates | `unisex`, `womens`, or `mens` |
| Add order-level `size_system` as a fallback | Catches any line you missed |
| Re-test against staging before deploying | Verify `resolved_size` in the response |

