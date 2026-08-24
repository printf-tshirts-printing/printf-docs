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
  - printf-rb
  - printf-go
---

# Quickstart

This guide takes you from zero to your first fulfilled order using order-api **2.4.0**. The API is RESTful and returns JSON. All requests require a bearer token in the `Authorization` header.

## Prerequisites

- A Printf account with at least one active facility (`facilityId`).
- An API key from the [developer dashboard](https://app.printf.dev/settings/api-keys).
- One design already uploaded (`designId`).

## Your first order

`size_system` is required as of 2.4.0. The example below uses `US`. Swap in `EU` or `JP` if your market requires it — a JP `XL` is 97 cm chest versus 112 cm for US, so the value you choose directly determines the garment that ships.

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

### Response

A successful `201` response includes `resolved_size` on every line, confirming the ladder that was applied:

```json
{
  "orderId": "ord_abc123",
  "status": "accepted",
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

The `X-Printf-Size-System: US` response header confirms the system applied at the order level.

**Verify `resolved_size.chest_cm` in every test.** It is the only field that proves the correct physical ladder was used.

## Required fields

Every order request must include:

| Field | Type | Notes |
|---|---|---|
| `accountId` | string | Your account identifier |
| `destination` | object | Full shipping address including `countryCode` |
| `lines[].designId` | string | Uploaded design identifier |
| `lines[].garmentSku` | string | Exact SKU — do not abbreviate to `sku` |
| `lines[].quantity` | integer | Units per line |
| `size_system` | string | `US`, `EU`, or `JP` — order or line level, required as of 2.4.0 |

## Error handling

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` provided and account routes to multiple facilities. Add `size_system`. |
| `size_system_implicit` | — | Warning: `size_system` was inferred. Becomes a `400` in 2.6. |

## Next steps

- [Sizing and fit](sizing.md) — full `size_system` and `fit` reference.
- [Bulk orders and templates](bulk-orders.md) — submit many orders at once and manage saved templates.

