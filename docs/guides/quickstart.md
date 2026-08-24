---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

This guide gets you from zero to a confirmed order in under five minutes.

## Prerequisites

- An API key from your [account dashboard](https://app.printf.dev/settings/api-keys)
- Your `accountId`, a `facilityId`, and at least one `designId`

## 1. Submit your first order

Send a `POST` to `/v2/orders`. All five of `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required. As of **2.4.0**, you must also supply `size_system`.

```bash
curl -X POST https://api.printf.dev/v2/orders \
  -H "Authorization: Bearer $PRINTF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
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
  }'
```

## 2. Read the response

A `201` response confirms the order. Each line includes a `resolved_size` object — check it before the order enters production:

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

The `X-Printf-Size-System: US` response header confirms the system used to resolve all lines in this order.

## 3. Handle errors

| Code | HTTP status | What to do |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to your request — your account routes to multiple facilities |
| `size_system_implicit` | — (warning) | Add `size_system` before 2.6, when this warning becomes a hard error |
| `unauthorized` | 401 | Check your API key |
| `invalid_request` | 400 | Check required fields: `accountId`, `destination`, `designId`, `garmentSku`, `quantity` |

## 4. Next steps

- [Sizing and fit](docs/guides/sizing.md) — full `size_system` resolution rules and ladder reference
- [Bulk orders and templates](docs/guides/bulk-orders.md) — high-volume submission and template migration
- [Webhooks](docs/guides/webhooks.md) — receive `resolved_size` in fulfillment events

