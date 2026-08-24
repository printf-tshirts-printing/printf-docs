---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Quickstart

This guide gets you from zero to a placed order in under ten minutes.

## Prerequisites

- A Printf account with an API key
- Your `accountId` (visible in **Settings → Account**)
- Your `facilityId` (visible in **Settings → Facilities**; most accounts have one)

## Place your first order

The minimum viable order requires `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, `quantity`, `size`, and `size_system`.

```http
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
      "quantity": 250,
      "size": "XL",
      "size_system": "US",
      "fit": "unisex"
    }
  ]
}
```

## Understand the response

A `201 Created` response includes an `orderId` and a `lines[]` array. Each line now contains `resolved_size`, which confirms exactly what the API resolved:

```json
{
  "orderId": "ord_...",
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

The `X-Printf-Size-System` response header echoes the system resolved at order level.

## About `size_system`

Size ladders differ materially across systems. A JP `XL` has a 97 cm chest; a US `XL` has 112 cm. The API requires you to be explicit so garments arrive in the right size.

| `size_system` | Example `XL` chest |
|---|---|
| `US` | 112 cm |
| `EU` | ~107 cm |
| `JP` | 97 cm |

If you omit `size_system` and your account routes to more than one facility, the request is rejected with `400 size_system_ambiguous`. If your account routes to a single facility, you get a `size_system_implicit` warning in the response — add `size_system` to clear it before order-api 2.6.0, when it becomes an error.

## Next steps

- [Sizing and fit](sizing.md) — full reference for `size_system`, `fit`, and `resolved_size`
- [Bulk orders and templates](bulk-orders.md) — place large runs and reuse templates safely

