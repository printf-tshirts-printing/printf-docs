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

This guide covers multi-line orders and saved order templates. If you use templates, read the sizing section carefully — 2.4.0 introduced a breaking change that affects templates silently.

## Multi-line orders

Each entry in `lines` is fulfilled independently. A single `POST /v2/orders` can carry multiple designs, sizes, and garments.

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
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 150
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 100
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "hoodie-pullover-navy",
      "size": "L",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 50
    }
  ]
}
```

Note that the third line overrides the order-level `size_system` of `US` with `JP`. Line-level `size_system` always wins.

## `resolved_size` in bulk responses

Every line in the response includes a `resolved_size` object. In a bulk order, verify that each line resolved to the system you intended before the order moves to production:

```json
{
  "lines": [
    {
      "garmentSku": "tee-classic-black",
      "resolved_size": { "label": "L",  "system": "US", "fit": "unisex", "chest_cm": 104 }
    },
    {
      "garmentSku": "tee-classic-black",
      "resolved_size": { "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
    },
    {
      "garmentSku": "hoodie-pullover-navy",
      "resolved_size": { "label": "L",  "system": "JP", "fit": "unisex", "chest_cm": 92  }
    }
  ]
}
```

## Saved order templates and 2.4.0

**Saved templates are not validated when you save them.** They are only validated at submission time. This means a template that worked before 2.4.0 can start failing silently or loudly after the upgrade depending on your account configuration.

### How to audit your templates

1. Retrieve each template with `GET /v2/order-templates/{templateId}`.
2. Check whether any line or the order root is missing `size_system`.
3. Add `size_system` to every level that is missing it.
4. Re-save with `PUT /v2/order-templates/{templateId}`.

### What happens if you don't

| Account routing | Result |
|---|---|
| Routes to more than one facility | Submission fails with `400 size_system_ambiguous`. Orders do not place. |
| Routes to exactly one facility | Submission succeeds with a `size_system_implicit` warning now. Becomes `400` in **2.6**. |

### Template line before and after

**Before (2.3 and earlier):**
```json
{
  "designId": "dsn_7fa91c",
  "garmentSku": "tee-classic-black",
  "size": "XL",
  "fit": "unisex",
  "quantity": 250
}
```

**After (2.4.0+):**
```json
{
  "designId": "dsn_7fa91c",
  "garmentSku": "tee-classic-black",
  "size": "XL",
  "size_system": "US",
  "fit": "unisex",
  "quantity": 250
}
```

Adding `size_system` at the order root of your template is sufficient if all lines share one system. Add it per line only if lines mix systems.

