---
title: Quickstart
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Quickstart

This page takes you to your first order. As of Orders API 2.4.0, every order must say which size system its `size` labels use. A bare `size` such as `XL` returns `400 size_system_ambiguous` if your account can route to more than one facility. If your account routes to a single facility, it succeeds with a `size_system_implicit` warning that becomes an error in 2.6.

## Place an order

`POST /v2/orders`

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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

| Field | Required | Notes |
|---|---|---|
| `accountId` | Yes | Your account. |
| `destination` | Yes | Where the order ships. |
| `designId` | Yes, per line | The design to print. |
| `garmentSku` | Yes, per line | The garment. |
| `quantity` | Yes, per line | Units on this line. |
| `size_system` | Yes in practice | `US`, `EU` or `JP`. Set it on the order, and on a line only to override the order value. |
| `size` | Per line | A label on the ladder named by `size_system`. |
| `fit` | Optional, per line | For example `unisex`. |

Use the same field names with any Printf client library: printf-js, printf-py, printf-go, printf-java or printf-rb.

## Check the size that was ordered

The ladders differ. A JP `XL` is 97 cm and a US `XL` is 112 cm. Each response line carries `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Responses also carry an `X-Printf-Size-System` header.

## If your order fails

| Code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | `400`. Your account can route to more than one facility, and a bare label cannot be resolved. | Add `size_system` to the order. |
| `size_system_implicit` | Warning. Your account routes to a single facility and the facility default was used. Becomes an error in 2.6. | Add `size_system` to the order. |

For the full resolution order, and for what to do with saved order templates, see [Sizing and fit](/guides/sizing) and [Migrating saved templates](/guides/sizing#migrating-saved-templates).

