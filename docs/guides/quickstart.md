---
title: Quickstart
section: guides
last_reviewed: 2026-08-20
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
---

# Quickstart

This guide creates your first order against order-api 2.4.0. If you are upgrading from an earlier integration, read the [migration note](#migration-size-system) before sending any requests.

## Prerequisites

- An active Printf account (`accountId`)
- An API key with `orders:write` scope
- The identifier of the design you want to print (`designId`)
- The SKU of the garment (`garmentSku`)

## Create an order

```http
POST /v2/orders
Authorization: Bearer <api_key>
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

Required fields on every request: `accountId`, `destination`, `designId` (per line), `garmentSku` (per line), `quantity` (per line).

### Response

HTTP `201`. Each response line includes `resolved_size`:

```json
{
  "orderId": "ord_...",
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

The `X-Printf-Size-System` response header echoes the system resolved for the order (here: `US`).

## Error: `400 size_system_ambiguous`

If your account can route to more than one facility and you omit `size_system` at every level, the API returns:

```json
{ "error": "size_system_ambiguous", "status": 400 }
```

Fix: add `size_system` to the order or to each line. See the [Sizing and fit guide](sizing.md) for the full resolution ladder.

## SDK note

`printf-js` does not yet support `size_system` or `fit` (`supportsSizeSystem=false`). Until an update ships, send requests directly over HTTP as shown above.

## Migration: size system

See the standalone [migration note](../MIGRATION-2.4.0.md) for a full before/after diff. The short version: add `"size_system": "US"` (or `"EU"` / `"JP"`) to every order or line, and add `"fit"` per line. Multi-facility accounts that omit it get a `400` today. Single-facility accounts get a warning that becomes a `400` in 2.6.
