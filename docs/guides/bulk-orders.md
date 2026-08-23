---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-23
owner: platform-docs
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

Bulk orders let you submit many lines or many orders in a single call. As of **2.4.0**, every line must carry `size_system` and `fit` — this applies equally to one-off orders, bulk payloads, and saved templates.

## Saved templates — action required before 2.6

> **Warning:** Templates store the payload you saved them with. They do not inherit new required fields automatically. If a template has lines without `size_system`, those lines will trigger a `size_system_implicit` warning now and a hard error in **2.6**. Open each template in the dashboard, add `size_system` to every line (and to the order level as a fallback), then save.

Templates are invisible from the API — you cannot list or patch them programmatically. The only path is the dashboard.

## Bulk order payload — 2.4.0 shape

Each order in a bulk request follows the same schema as a single order. The fields `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every order object.

```json
POST /v2/orders/bulk
{
  "orders": [
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
}
```

## Per-line vs. order-level `size_system`

For events with mixed international attendee kits, set `size_system` at the line level so different lines can use different ladders:

```json
"lines": [
  {
    "designId": "dsn_7fa91c",
    "size": "XL",
    "size_system": "US",
    "fit": "unisex",
    "quantity": 150,
    "garmentSku": "tee-classic-black"
  },
  {
    "designId": "dsn_7fa91c",
    "size": "XL",
    "size_system": "JP",
    "fit": "unisex",
    "quantity": 100,
    "garmentSku": "tee-classic-black"
  }
]
```

A JP `XL` (97 cm chest) and a US `XL` (112 cm chest) are different garments. Mixing systems without explicit line-level values will produce the wrong garments for part of the run.

## `resolved_size` in bulk responses

Every line in a bulk response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Diff the `resolved_size.system` against your intent before confirming production runs. This is the only way to catch an implicit resolution before garments are cut.

## Error reference

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account sent a bare size. Add `size_system` to every affected line. |
| `size_system_implicit` | — (warning) | Single-facility account omitted `size_system`. Becomes a hard error in **2.6**. |

## Checklist before a bulk run

- [ ] Every line has `size_system` set explicitly
- [ ] Every line has `fit` set explicitly
- [ ] All saved templates have been updated in the dashboard
- [ ] `resolved_size` values have been reviewed in the staging response

