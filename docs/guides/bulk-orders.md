---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - POST /v2/orders/bulk
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers submitting multiple orders in a single call and using saved order templates. If you use templates, read the [saved templates](#saved-templates-and-size_system) section before upgrading to order-api 2.4.0 — templates are not visible from the API but they carry size fields that the new validation rules apply to.

## Submitting a bulk order

Send an array of order objects to `POST /v2/orders/bulk`. Each object follows the same schema as a single `POST /v2/orders` call, including the 2.4.0 `size_system` and `fit` requirements.

```json
POST /v2/orders/bulk
[
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
]
```

`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every order object in the array. A single invalid object causes the entire bulk request to be rejected with a `400` that identifies the failing index.

## Size system in bulk requests

You can set `size_system` once at the order level as a default for all lines in that order, or set it per line to mix systems within a single order. You cannot set a single `size_system` for the entire bulk array — each order object carries its own.

If your bulk submission targets accounts that route to multiple facilities, every order object must supply `size_system` at the order or line level. Orders that omit it are rejected with `400 size_system_ambiguous`.

## Saved templates and `size_system`

Saved order templates are stored server-side and are not returned by any list endpoint. If you created templates before 2.4.0, they do not contain `size_system` or `fit` fields.

**What happens when you submit a template-based order after 2.4.0:**

- If your account is **single-facility**, the facility default fills in `size_system`. You receive a `size_system_implicit` warning. This resolves correctly today but **becomes a `400` error in 2.6**.
- If your account is **multi-facility**, the submission is rejected immediately with `400 size_system_ambiguous`.

**How to fix your templates:**

You cannot patch a template through the API. You must recreate it with `size_system` (and optionally `fit`) set on every line, or at the template's order level. Contact support (reference code `size_system_implicit`) if you need to enumerate templates associated with your account.

## `resolved_size` in bulk responses

Each order object in the bulk response includes `resolved_size` on every line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Check `chest_cm` against your specification sheet when running bulk production jobs. A JP `XL` (97 cm) and a US `XL` (112 cm) are both valid but differ by a full size step.

## Error reference for bulk orders

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Order routes to multiple facilities; `size_system` required |
| `size_system_implicit` | — (warning) | `size_system` resolved from facility default; becomes `400` in 2.6 |
| `bulk_partial_failure` | 207 | One or more orders in the array failed; see per-order `errors` array |

