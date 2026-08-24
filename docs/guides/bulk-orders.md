---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
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

This guide covers placing large orders and working with saved order templates. If you are on order-api **2.3 or earlier**, read the [migration note](#migration-from-23) before using templates — `size_system` is now required and templates are a common place it goes missing.

## Placing a bulk order

Bulk orders use the same `POST /v2/orders` endpoint as single-item orders. All required fields apply regardless of line count.

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The order-level `size_system` is the default for every line. You can override it per line by setting `size_system` on the line itself — useful when a single order mixes regional sizing.

## Required fields reminder

Every request must include `accountId`, `destination`, and at least one line. Every line must include `designId`, `garmentSku`, and `quantity`. Missing any of these returns `400 missing_required_field`.

## Reading resolved_size

Each line in the response includes `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

For bulk runs, validate `resolved_size.chest_cm` programmatically before surfacing the order summary to buyers. A JP XL (97 cm) and a US XL (112 cm) are not the same garment.

## Working with saved order templates

Templates are stored server-side and submitted through `POST /v2/orders` at execution time. They are **not returned by any list endpoint**, which makes them easy to forget during upgrades.

### Auditing your templates for 2.4.0 compatibility

Every template that omits `size_system` at order level and per line will hit one of these errors after 2.4.0:

| Account type | Error | Blocking? |
|---|---|---|
| Multi-facility | `400 size_system_ambiguous` | Yes, immediate |
| Single-facility | `size_system_implicit` warning | No, but becomes `400 size_system_required` in 2.6 |

Steps to audit:

1. Pull your saved templates from whatever internal store or config management system you use.
2. For each template, confirm `size_system` is set at the order level or on every line.
3. Update and redeploy. There is no API endpoint to validate a template in isolation — test it against the staging environment.

### Template example (after update)

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

## Migration from 2.3

See the [full migration note](../migration-notes/2.4.0.md) for before/after payloads. Short version: add `size_system` at order level and on every line. Templates are the most common miss.

