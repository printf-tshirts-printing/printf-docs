---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

This guide covers submitting large orders and using saved order templates. Both are affected by the sizing changes introduced in **order-api 2.4.0**.

## Submitting a bulk order

Bulk orders follow the same shape as single orders. Each line item requires `designId`, `garmentSku`, `quantity`, `size`, and — starting in 2.4.0 — you should always include `size_system` and `fit`.

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
      "quantity": 150,
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
    }
  ]
}
```

The response for each line includes `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Check `chest_cm` against your expected measurement for each size. A mismatch means the wrong `size_system` was applied.

## Saved order templates

> **Action required if you have saved templates.** Templates are not visible through the API, but they are affected by this change.

Templates created before 2.4.0 have no `size_system` or `fit` fields. When a template is submitted:

- If your account routes to a **single facility**, the order is accepted with a `size_system_implicit` warning. The resolved system will be that facility's default — verify it matches your intent.
- If your account routes to **multiple facilities**, the order is rejected with `400 size_system_ambiguous`.

To fix a template, retrieve it, add `size_system` and `fit` to each line (and optionally `size_system` at the order level), and save it back. There is no bulk-update endpoint for templates; each template must be updated individually.

## Mixed size systems in one order

You can mix size systems within a single order by setting `size_system` per line and omitting or overriding the order-level value:

```json
"lines": [
  {
    "designId": "dsn_7fa91c",
    "size": "L",
    "size_system": "US",
    "fit": "unisex",
    "quantity": 100,
    "garmentSku": "tee-classic-black"
  },
  {
    "designId": "dsn_7fa91c",
    "size": "L",
    "size_system": "JP",
    "fit": "womens",
    "quantity": 50,
    "garmentSku": "tee-classic-black"
  }
]
```

The two `"L"` lines above produce different garments. Check `resolved_size.chest_cm` on each line in the response.

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and `size_system` is missing |
| `size_system_implicit` | — (warning) | System resolved from facility default; becomes an error in 2.6 |
