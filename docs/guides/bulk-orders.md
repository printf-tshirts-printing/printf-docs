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

Bulk orders and saved templates follow the same `POST /v2/orders` contract as single orders. As of **2.4.0**, that contract requires explicit size system information. This guide covers what changed and how to update your templates and batch scripts.

## What changed in 2.4.0

Every line in a bulk order must resolve to an unambiguous size system. See [Sizing and fit](sizing.md) for the full resolution chain. The short version:

- Add `size_system` at the order level, the line level, or both.
- Multi-facility accounts that omit `size_system` receive `400 size_system_ambiguous`.
- Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning today; it becomes a hard error in **2.6**.

## Bulk order example

A complete bulk order with multiple lines, each with an explicit `size_system` and `fit`:

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
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The order-level `size_system` acts as a default for any line that omits it. Setting it on each line explicitly is safer for bulk payloads built programmatically.

## Saved templates

**Templates are not visible in the API response list.** They are stored server-side and evaluated at submission time. If you saved a template before 2.4.0, it almost certainly omits `size_system`.

How to find and fix them:

1. Open the Printf dashboard → **Templates**.
2. Open each template and check every line for a `size_system` value.
3. Add `size_system` at the order level, the line level, or both.
4. Save the updated template.

A template that still omits `size_system` will:

- Return `size_system_implicit` warnings from **2.4.0** (single-facility accounts).
- Return `400 size_system_ambiguous` from **2.4.0** (multi-facility accounts).
- Become a hard error for **all accounts** in **2.6**.

## `resolved_size` in bulk responses

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For large orders, cross-reference `resolved_size.system` against your expected system before approving production. A mismatch here means garments in the wrong size.

## SDK support

All official SDKs (`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb`) expose `size_system` and `fit` as typed fields in their 2.4.x releases. Upgrade before sending bulk orders to avoid untyped-dict workarounds.

