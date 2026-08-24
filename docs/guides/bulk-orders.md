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
  - printf-rb
  - printf-go
---

# Bulk orders and templates

This guide covers submitting multiple orders in a single request and using saved order templates. **If you use templates, read the [size system section](#size-system-and-templates) before upgrading to order-api 2.4.0** — templates store line-level fields and will continue to submit without `size_system` until 2.6, at which point they become hard errors.

## Bulk order structure

A bulk order is an array of standard order objects. Each order must include all required fields: `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`.

As of **2.4.0**, include `size_system` at the order level, the line level, or both:

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

## Size system and templates

Saved order templates are the most common source of `size_system_ambiguous` errors after a version upgrade, because templates are invisible from the API and customers forget they exist until an order arrives in the wrong size.

**What to do right now:**

1. List every saved template in your account dashboard.
2. Open each template and confirm whether `size_system` is set at the order or line level.
3. If it is missing, add it. The value should match the market the template was created for (`US`, `EU`, or `JP`).

Templates that omit `size_system` will:

- Emit `size_system_implicit` (warning) through the 2.5.x release window if the account routes to a single facility.
- Return `400 size_system_ambiguous` immediately if the account can route to more than one facility.
- Return `400 size_system_ambiguous` for all accounts starting in **2.6**.

## `resolved_size` in bulk responses

Every line in the bulk response includes `resolved_size`, giving you the ladder that was applied per line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk orders mixing US and JP lines, `chest_cm` is the fastest way to confirm each line resolved to the correct physical garment. A JP `XL` will show `97`; a US `XL` will show `112`.

## Error and warning reference

| Code | HTTP status | When it fires |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` absent, multi-facility account or 2.6+ |
| `size_system_implicit` | — (warning) | `size_system` absent, single-facility account, pre-2.6 |

For the full size-system reference, see the [Sizing and fit](sizing.md) guide.

