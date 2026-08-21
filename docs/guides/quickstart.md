---
title: Quickstart
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

This guide gets you from zero to a fulfilled order in the fewest steps. It uses **order-api 2.4.0**, which requires `size_system` and `fit` on every line.

## Prerequisites

- An `accountId` (format: `acct_…`)
- An API key in the `Authorization` header
- A design ID (format: `dsn_…`) from the design API

## Your first order

Send a `POST /v2/orders` request with `accountId`, `destination`, and at least one line. Each line requires `designId`, `garmentSku`, `quantity`, `size`, `size_system`, and `fit`.

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

### What comes back

The response includes `resolved_size` on every line:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

The response header `X-Printf-Size-System` tells you the system applied to the whole order.

Verify `chest_cm` before sending to fulfillment. If it does not match your expected measurement, check that `size_system` on the line matches your intent.

## Required fields at a glance

| Field | Level | Required |
|---|---|---|
| `accountId` | Order | ✅ Always |
| `destination` | Order | ✅ Always |
| `size_system` | Order | Recommended (see below) |
| `facilityId` | Order | Recommended |
| `designId` | Line | ✅ Always |
| `garmentSku` | Line | ✅ Always |
| `quantity` | Line | ✅ Always |
| `size` | Line | ✅ Always |
| `size_system` | Line | Recommended (see below) |
| `fit` | Line | Recommended |

## If you omit size_system

The API resolves it: line → order → account → facility default. If your account can route to more than one facility, the request is rejected with `400 size_system_ambiguous`. Add `size_system` to your request.

If your account routes to only one facility, the request succeeds with a `size_system_implicit` warning in the response. This warning becomes a hard error in **2.6**.

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full resolution rules, size system tables, and error reference
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line orders and updating saved templates
