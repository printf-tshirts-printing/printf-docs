---
title: Quickstart
section: guides
last_reviewed: 2026-08-21
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

This guide walks you from zero to a submitted order in under ten minutes.

## Before you start

You need:

- A Printf account identifier (`accountId`)
- An API key — generate one in the dashboard under **Settings → API keys**
- A design identifier (`designId`) — upload a design in the dashboard to get one
- A garment SKU (`garmentSku`) — browse the product catalog for available values

## Your first order

Send a `POST` to `/v2/orders`. All five fields — `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` — are required. `size_system` is required as of **2.4.0**; omitting it on a multi-facility account returns `400 size_system_ambiguous`.

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## What you get back

A successful submission returns the created order. Each line now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response also includes the `X-Printf-Size-System` header confirming the system applied.

Check `resolved_size.chest_cm` against your design spec. A US `XL` chest is 112 cm; a JP `XL` chest is 97 cm. If the wrong system was applied, cancel the order and resubmit with the correct `size_system`.

## Choosing a size system

| `size_system` | Region | US XL chest |
|---|---|---|
| `US` | United States | 112 cm |
| `EU` | Europe | 107 cm |
| `JP` | Japan | 97 cm |

Set `size_system` at the order level as a default for all lines. You can override it per line if an order spans multiple systems.

## Required fields at a glance

| Field | Required | Notes |
|---|---|---|
| `accountId` | ✅ | Your account identifier |
| `destination` | ✅ | Full shipping address object |
| `lines[].designId` | ✅ | Design to print |
| `lines[].garmentSku` | ✅ | Garment to print on |
| `lines[].quantity` | ✅ | Units to produce |
| `size_system` | ✅ (2.4.0+) | Order-level or per-line; `US`, `EU`, or `JP` |
| `lines[].size` | ✅ | Label within the chosen system (e.g. `XL`) |
| `lines[].fit` | recommended | `unisex` is most common |

## Common errors

| Error code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | Multi-facility account, `size_system` absent | Add `size_system` to the order or each line |
| `size_system_implicit` | Single-facility account, `size_system` absent (warning; error in 2.6) | Add `size_system` now |

## Next steps

- **[Sizing and fit](sizing.md)** — size ladders, `fit` values, `resolved_size` in detail
- **[Bulk orders and templates](bulk-orders.md)** — multiple lines, saved templates, and how to audit templates for the 2.4.0 requirement
- **API reference** — `POST /v2/orders` and `GET /v2/orders/{orderId}` in `docs/api/`
