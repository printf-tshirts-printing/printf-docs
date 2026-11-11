---
title: Quickstart
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js]
---

# Quickstart

This page takes you to your first order with `POST /v2/orders`.

Breaking change in order-api 2.4.0: `size` is no longer interpreted without a size system. Include `size_system` in every order, as the example below does. If you copied an earlier version of this example, your request has a bare `size` and will be rejected with `400 size_system_ambiguous` or warn with `size_system_implicit`.

## Create an order

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
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## Required fields

| Field | Where | Notes |
|---|---|---|
| `accountId` | Order | Required |
| `destination` | Order | Required |
| `designId` | Each line | Required |
| `garmentSku` | Each line | Required |
| `quantity` | Each line | Required |
| `size_system` | Order and each line | `US`, `EU` or `JP`. Send it on the order at minimum. |
| `fit` | Each line | Optional |

## Read the result

Each line in the response carries `resolved_size`. This is the size Printf will actually fulfill. The response also carries an `X-Printf-Size-System` header.

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

A JP `XL` is 97 cm and a US `XL` is 112 cm, so check this on your first order.

## Errors you may see

| Code | Meaning |
|---|---|
| `size_system_ambiguous` | `400`. Your account can route to more than one facility and no `size_system` was resolvable. |
| `size_system_implicit` | Warning. The order was created, but its size system was inferred. This becomes an error in 2.6. |

## Next

- [Sizing and fit](/guides/sizing): how bare sizes resolve and how to migrate saved templates.
- [Bulk orders and templates](/guides/bulk-orders): many lines in one order.

