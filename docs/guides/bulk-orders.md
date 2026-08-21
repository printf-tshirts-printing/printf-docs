---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
ccovers_sdks:
  - printf-js
  - printf-py
---

# Bulk orders and templates

As of order-api **2.4.0**, saved order templates and bulk order payloads that omit `size_system` will produce deprecation warnings or hard errors. This page tells you what to check and what to change.

> **SDK note:** `printf-js` and `printf-py` do not yet model `size_system` or `fit`. Use raw HTTP or pass untyped extra fields until SDK releases ship.

## Why templates are high-risk

Templates are not validated when they are saved. A template that omits `size_system` will:

- **Today (2.4.0):** succeed on single-facility accounts with a `size_system_implicit` warning; fail with `400 size_system_ambiguous` on multi-facility accounts.
- **In 2.6:** fail for all accounts.

Templates are invisible from the API list view. Audit them now.

## Auditing your templates

1. Retrieve each template and inspect its `lines` array.
2. For every line, confirm `size_system` is present either at the line level or at the order level.
3. Confirm `fit` is present on every line.

## Updated template schema

Before (2.3.x and earlier):

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
      "size": "XL",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

After (2.4.0 and later):

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## Bulk payloads

For bulk submissions, add `size_system` at the order level as a default and override per line when orders span multiple systems. The `resolved_size` object on each response line confirms what was applied:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

## Error and warning reference

| Code | Status | When it fires |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system`; account routes to multiple facilities. |
| `size_system_implicit` | warning | No `size_system`; single-facility account. Becomes error in 2.6. |

