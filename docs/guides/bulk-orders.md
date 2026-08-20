---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-20
owner: dx
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
---

# Bulk orders and templates

## Breaking change in 2.4.0 — saved templates must be updated

If you use **saved order templates**, read this section before your next order run.

order-api 2.4.0 introduces `size_system` to resolve bare size labels (`L`, `XL`, etc.) against a named regional ladder. Templates saved before this release almost certainly do not include `size_system`. The impact depends on how many fulfillment facilities your account can route to:

| Account routing | Result when `size_system` is absent |
|---|---|
| Single facility | Warning `size_system_implicit` — order fulfills, but **becomes a hard error in 2.6** |
| Multiple facilities | **`400 size_system_ambiguous` — order is rejected immediately** |

Templates are not visible through the API. You must audit them in the Printf dashboard or contact support (reference: `size_system_ambiguous`) to list all templates for your account.

## How to update your templates

Add `size_system` at the order root and on each line. Also add `fit` per line.

**Before (breaks for multi-facility accounts)**

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

**After (safe for all accounts)**

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

## SDK compatibility

`printf-js` (v2.3.4) and `printf-py` (v2.3.2) do not yet support `size_system` (`supportsSizeSystem: false`). Use raw HTTP calls for all bulk jobs that include `size_system` until updated SDK releases ship.

## Validating resolved sizes in bulk runs

Every response line now includes `resolved_size`:

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

Log and verify `chest_cm` values on the first run after updating templates. Ladder measurements differ materially across systems (JP `XL` = 97 cm; US `XL` = 112 cm). A wrong `size_system` produces a valid order with wrong garment dimensions — not an error.

## Error reference

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` omitted on multi-facility account |
| `size_system_implicit` | — (warning) | `size_system` resolved from facility default; becomes a hard error in 2.6 |

