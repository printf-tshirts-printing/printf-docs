---
title: Quickstart
section: Guides
last_reviewed: 2026-08-21
owner: platform-docs
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

This guide walks you through placing your first order with the Printf API. Examples target **API 2.4.0**.

## Prerequisites

- An account ID (`accountId`) — find it in the dashboard under **Settings → Account**.
- A design ID (`designId`) — upload a design in the dashboard to get one.
- A garment SKU (`garmentSku`) — browse available SKUs in the Product Catalog.

## Place an order

```http
POST /v2/orders
Authorization: Bearer <your_api_key>
Content-Type: application/json
```

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

### What's required

| Field | Level | Notes |
|---|---|---|
| `accountId` | Order | Your account identifier |
| `destination` | Order | Full shipping address |
| `designId` | Line | The design to print |
| `garmentSku` | Line | The garment to print on |
| `quantity` | Line | Number of units |

### What's new in 2.4.0

| Field | Level | Notes |
|---|---|---|
| `size_system` | Order and/or line | `US`, `EU`, or `JP`. Set it — omitting it will be an error in 2.6. |
| `fit` | Line | `unisex`, `womens`, `mens` |

## Read the response

On success you receive `201 Created`. Each line includes `resolved_size` confirming what the API resolved:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response header `X-Printf-Size-System` shows the system used to resolve bare labels.

## Errors to know

| Code | Meaning |
|---|---|
| `size_system_ambiguous` | Your account routes to multiple facilities and no `size_system` was provided. Add `size_system` to the order or line. |
| `size_system_implicit` | Warning: bare label resolved via facility default. Will be an error in 2.6. |

See [Sizing and fit](sizing.md) for the full resolution rules.

