---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, POST /v2/order-templates, POST /v2/order-templates/{templateId}/submit
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Bulk orders and templates

This guide covers sending large single orders and working with saved order templates.

> **2.4.0 breaking change — templates require `size_system`.**
> Saved templates that omit `size_system` will fail or warn depending on your account's facility routing. See [Sizing and fit](sizing.md) for the full resolution ladder. Review and update your templates before 2.6.0, when bare size labels become an error.

## Bulk orders

Send many units in a single request by adding multiple items to `lines[]`. Each line requires `designId`, `garmentSku`, `quantity`, `size`, and `size_system`. All lines share the order-level `destination` and `accountId`.

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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
      "quantity": 250,
      "size": "XL",
      "size_system": "US",
      "fit": "unisex"
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 100,
      "size": "L",
      "size_system": "US",
      "fit": "unisex"
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 50,
      "size": "M",
      "size_system": "EU",
      "fit": "womens"
    }
  ]
}
```

You can mix `size_system` values across lines. A per-line `size_system` always overrides the order-level one.

## Saved order templates

Templates store a full order payload that you can resubmit on demand. As of 2.4.0, any template saved without `size_system` will trigger `size_system_implicit` warnings (single-facility accounts) or be rejected outright with `400 size_system_ambiguous` (multi-facility accounts).

**Templates are not visible from the API listing — you must update each one by identifier.** If you created templates before 2.4.0, retrieve them now and add `size_system` to the order level, to each line, or both.

### Template line schema (2.4.0 and later)

| Field | Required | Notes |
|---|---|---|
| `designId` | ✅ | |
| `garmentSku` | ✅ | |
| `quantity` | ✅ | |
| `size` | ✅ | Bare label; must be paired with `size_system`. |
| `size_system` | ✅ (recommended) | `US`, `EU`, or `JP`. Required in 2.6.0. |
| `fit` | Optional | `unisex`, `mens`, `womens`. |

### `resolved_size` in template submissions

When you submit a template, each line in the response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Verify `resolved_size.system` and `resolved_size.chest_cm` match your expectation before treating a bulk submission as confirmed.

## Errors and warnings

| Code | Type | Meaning |
|---|---|---|
| `size_system_ambiguous` | `400` error | No `size_system` resolved; account routes to multiple facilities. |
| `size_system_implicit` | Warning | `size_system` inferred from single facility. Becomes `400` in 2.6.0. |

