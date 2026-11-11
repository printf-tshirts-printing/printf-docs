---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py]
---

# Bulk orders and templates

Breaking change in order-api 2.4.0: orders with many lines, and orders placed from saved templates, must state a size system. A bare `size` with no resolvable system is rejected with `400 size_system_ambiguous` on accounts that can route to more than one facility. On single-facility accounts it produces a `size_system_implicit` warning, which becomes an error in 2.6.

## Bulk orders

A bulk order is a single `POST /v2/orders` with several entries in `lines`. Set `size_system` once on the order. Override it on a line only when that line uses a different system.

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
      "size": "XL",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 40,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The two lines above both say `XL` but are different garments. The first resolves to US (chest 112 cm) from the order. The second is set to JP on the line and resolves to chest 97 cm.

`accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

### Precedence

| Order | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order |
| 3 | The account's size system |
| 4 | The fulfilling facility's default |

Rule 4 depends on routing, not on your payload. If your account can route to more than one facility, a request that would reach rule 4 fails with `size_system_ambiguous`.

### Check what was used

Each response line carries `resolved_size`, and webhook payloads carry it too. Compare it with what you intended before the order is fulfilled.

```json
{ "label": "XL", "system": "JP", "fit": "unisex", "chest_cm": 97 }
```

## Saved order templates

Templates created before 2.4.0 carry no explicit `size_system`. They are not visible from the API, and they predate this change, so they resolve against whatever the rules above produce today.

:::warning
A bulk order placed from an old template can go through at the wrong size on a single-facility account. The `size_system_implicit` warning is easy to miss in an automated flow.
:::

For each saved template:

1. Open the template and decide which size system its sizes were written for.
2. Set `size_system` on the template's order. Set it on individual lines only where they differ.
3. Save the template.
4. Place a test order from it and check `resolved_size` on each line.

The full set of before/after payloads is in [Sizing and fit](/guides/sizing#migrating-saved-templates).

| Account type | Old template, no change | From |
|---|---|---|
| Can route to more than one facility | `400 size_system_ambiguous` | 2.4.0 |
| Routes to a single facility | `size_system_implicit` warning | Error in 2.6 |

