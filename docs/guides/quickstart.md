---
title: Quickstart
section: guides
last_reviewed: 2026-08-20
owner: dx
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
---

# Quickstart

This guide gets you from zero to a submitted order in under five minutes using the `printf-js` SDK.

> **SDK notice — size_system support pending.** `printf-js` does not yet support `size_system` natively (current release: v2.3.4, `supportsSizeSystem: false`). Use the raw HTTP examples below when `size_system` is required, or pin your SDK until a new release ships. Raw HTTP always reflects the current API contract.

## 1. Install

```bash
npm install printf-js
```

## 2. Create an order

The `size_system` field tells Printf which regional size ladder to apply. Set it at the order level as a default and override it per line when you mix regions.

**HTTP**

```http
POST /v2/orders
Content-Type: application/json

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

**Response** (HTTP 201)

```json
{
  "orderId": "ord_abc123",
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "resolved_size": {
        "label": "XL",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 112
      }
    }
  ]
}
```

The `X-Printf-Size-System: US` response header confirms the effective size system used to resolve the order. Check it whenever you omit `size_system` at the order level.

### Resolution order for bare size labels

When `size_system` is omitted, Printf resolves it in this order:

| Priority | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order root |
| 3 | `size_system` on the account |
| 4 | Fulfilling facility default |

If your account can route to **more than one facility** and no `size_system` is set anywhere, the request is rejected with `400 size_system_ambiguous`. Set `size_system` explicitly to avoid this.

## 3. Error reference

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; `size_system` must be explicit |
| `size_system_implicit` | — | Warning: `size_system` resolved from facility default. Becomes a hard error in 2.6 |

## 4. Next steps

- [Sizing and fit guide](docs/guides/sizing.md) — regional ladder tables, chest measurements, fit options
- [Orders API reference](docs/api/orders.md) — full field reference
- [Bulk orders and templates](docs/guides/bulk-orders.md) — if you use saved templates, read this before your next run

