---
title: Quickstart
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

# Quickstart

Place your first order with the Printf API in under five minutes.

## Prerequisites

- An API key from the [Printf dashboard](https://app.printf.dev/settings/api-keys)
- Your `accountId` (shown on the account overview page)
- A `designId` for an approved design
- A `garmentSku` for an available product
- A `facilityId` for your fulfillment facility

## Place an order

Send a `POST` to `/v2/orders`. `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every request.

**As of 2.4.0, `size_system` is required.** Omitting it is deprecated for single-facility accounts and a hard error for multi-facility accounts. Include it from the start.

```json
POST /v2/orders
Authorization: Bearer <your-api-key>
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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

## Read the response

A successful response is `201 Created`. Each line includes a `resolved_size` object so you can confirm what Printf will produce:

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

The response also includes an `X-Printf-Size-System` header echoing the resolved system:

```
X-Printf-Size-System: US
```

## Common first-order errors

| Code | HTTP status | Cause | Fix |
|---|---|---|
|---|
| `size_system_ambiguous` | `400` | `size_system` omitted and account routes to more than one facility. | Add `size_system` to the order or the affected line. |
| `size_system_implicit` | — (warning in 2.4, error in 2.6) | `size_system` omitted and account routes to exactly one facility. | Add `size_system` to suppress the warning now and avoid a future `400`. |

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full `size_system`, `fit`, and `resolved_size` reference
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line orders and how to audit saved templates

