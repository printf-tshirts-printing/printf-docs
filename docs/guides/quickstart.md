---
title: Quickstart
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Quickstart

This guide takes you to your first order with `POST /v2/orders`.

Breaking change in order-api 2.4.0: every order needs an explicit `size_system` (`US`, `EU` or `JP`). Without it, accounts that can route to more than one facility get `400 size_system_ambiguous`. Single-facility accounts get a `size_system_implicit` warning that becomes an error in 2.6.

## 1. Build the order

These fields are required on every request:

| Field | Where |
|---|---|
| `accountId` | Order |
| `destination` | Order |
| `designId` | Each line |
| `garmentSku` | Each line |
| `quantity` | Each line |

Also send `size_system` on the order, and `size` and `fit` on each line.

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

:::warning
A US `XL` is 112 cm chest and a JP `XL` is 97 cm. Always send `size_system`, or the size may resolve to a system you did not mean.
:::

## 2. Send it

Send the payload to `POST /v2/orders` with any of the Printf client libraries: `printf-js`, `printf-py`, `printf-go`, `printf-java` or `printf-rb`. All five are generated from the same OpenAPI document, so the fields above are the same in each.

## 3. Check what was ordered

Each line in the response carries `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

The response also carries an `X-Printf-Size-System` header, and webhook payloads for the order include `resolved_size`. If `chest_cm` is not what you intended, fix the payload before the order is fulfilled.

## Errors you might hit

| Code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | `400`. Your account can route to more than one facility and a size had no system. | Send `size_system` on the order or the line. |
| `size_system_implicit` | Warning. The size resolved to the facility default. Becomes an error in 2.6. | Send `size_system`. |

How bare sizes resolve, and how to update saved order templates, is covered in [Sizing and fit](/guides/sizing). Mixed-system and large orders are covered in [Bulk orders and templates](/guides/bulk-orders).

