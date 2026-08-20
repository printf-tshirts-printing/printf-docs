---
title: Quickstart
section: guides
last_reviewed: 2026-08-20
owner: docs-platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
---

# Quickstart

This guide gets you from zero to a confirmed order in under ten minutes. It uses the REST API directly; SDK equivalents follow each request.

> **printf-js / printf-py / printf-go / printf-java / printf-rb are not yet updated for 2.4.0.** All five report `supportsSizeSystem=false`. Use raw HTTP or wait for the SDK releases before relying on the new size fields through a client library.

## Prerequisites

- An API key with `orders:write` scope.
- Your `accountId` (shown on the Account page in the dashboard).
- Your facility ID (`facilityId`). Find it under **Settings → Fulfillment**.

## Create an order

```http
POST /v2/orders
Authorization: Bearer <api_key>
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

Required fields on every request: `accountId`, `destination`, `designId`, `garmentSku`, `quantity`.

`size_system` is strongly recommended on every request. Omitting it triggers a `size_system_implicit` deprecation warning today and will become a hard error in **2.6.0**.

## Understand the response

Each line item in the response now includes `resolved_size`:

```json
{
  "id": "ord_abc123",
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

The response also includes the header `X-Printf-Size-System: US` confirming which size system was applied to the order.

> **Ladder note:** a JP `XL` is 97 cm chest; a US `XL` is 112 cm. Always verify `resolved_size.chest_cm` in test before going to production with a new size system.

## Size system resolution order

When `size_system` is not set on a line, Printf resolves it in this order:

| Priority | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order |
| 3 | Default set on the account |
| 4 | Default of the fulfilling facility |

If your account routes to **more than one facility** and resolution reaches step 4, the request is rejected with `400 size_system_ambiguous`. Set `size_system` explicitly to avoid this.

## Next steps

- [Sizing and fit guide](sizing.md) — full ladder tables for US, EU, and JP.
- [Webhooks](webhooks.md) — `resolved_size` also appears on webhook payloads.
- [Bulk orders and templates](bulk-orders.md) — if you use saved templates, read the migration note before 2.6.0.

