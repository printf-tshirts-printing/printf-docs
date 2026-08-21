---
title: Quickstart
section: guides
last_reviewed: 2026-08-21
owner: dx
covers_endpoints: ["POST /v2/orders"]
covers_sdks: [printf-js, printf-py]
---

# Quickstart

This guide gets you from zero to a confirmed order in under five minutes.

## Before you start

You need:

- A Printf account with an `accountId`
- An API key in the `Authorization: Bearer <key>` header
- At least one design uploaded (gives you a `designId`)

## Create your first order

Post to `/v2/orders`. The example below is a complete, working request.

```http
POST /v2/orders HTTP/1.1
Content-Type: application/json
Authorization: Bearer <your_key>

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

> **`size_system` is not optional in 2.4.0 if your account can route to more than one facility.** Omitting it on a multi-facility account returns `400 size_system_ambiguous`. Add it now — it becomes a hard error for all accounts in 2.6.

### Required fields

| Field | Level | Notes |
|---|---|---|
| `accountId` | Order | Your account identifier |
| `destination` | Order | Full address object |
| `designId` | Line | Design to print |
| `garmentSku` | Line | Blank garment reference |
| `quantity` | Line | Units to produce |

### Reading the response

Every line in the response now includes `resolved_size`:

```json
{
  "lines": [
    {
      "label": "XL",
      "system": "US",
      "fit": "unisex",
      "chest_cm": 112
    }
  ]
}
```

The response header `X-Printf-Size-System` echoes the effective size system for the whole order. If it differs from what you expected, check your account's facility routing.

## Next steps

- [Sizing and fit](sizing.md) — full size ladder tables for US, EU, and JP
- [Webhooks](webhooks.md) — `resolved_size` also appears on fulfillment events
- [Bulk orders and templates](bulk-orders.md) — update saved templates before 2.6

