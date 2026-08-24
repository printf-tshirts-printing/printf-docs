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

This guide walks you through placing your first order with the Printf API. It reflects **order-api 2.4.0**. If you are running an older client library (`printf-js`, `printf-py`, `printf-java`, `printf-go`, or `printf-rb`), upgrade to the 2.4.x release before following these steps — the `size_system` field is required in 2.4.0 and missing it will produce a `size_system_ambiguous` error on multi-facility accounts.

## Before you start

You need:

- An API key bound to an `accountId`
- A `designId` for the artwork you want to print
- A `garmentSku` for the garment you want to print on
- The destination address

## Place your first order

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

`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every request. `size_system` is required in practice — omitting it produces a warning on single-facility accounts and an error on multi-facility accounts.

## Read the response

A `201 Created` response includes a `resolved_size` object on each line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Check `resolved_size.system` to confirm the API interpreted your size label the way you intended. The `X-Printf-Size-System` response header contains the same system value at the order level.

## Size systems

Printf supports three size systems:

| `size_system` | Region | `XL` chest |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 107 cm |
| `JP` | Japan | 97 cm |

Set `size_system` at the order level for uniform orders, or per line when mixing systems.

## Next steps

- **[Sizing and fit](sizing.md)** — full size system reference, fit options, and per-line overrides
- **[Bulk orders and templates](bulk-orders.md)** — sending multiple lines and managing saved templates
- **API reference** — `POST /v2/orders`, `GET /v2/orders/{orderId}`
