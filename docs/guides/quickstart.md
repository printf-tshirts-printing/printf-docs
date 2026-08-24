---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

This guide takes you from zero to a confirmed order in under ten minutes.

## Prerequisites

- An account ID (`acct_…`) from the dashboard
- An API key with `orders:write` scope
- A design ID (`dsn_…`) you have already uploaded

## 1 — Place your first order

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

`size_system` is **required** as of 2.4.0. Omitting it on a multi-facility account returns `400 size_system_ambiguous`.

## 2 — Read the response

A successful `201` response includes a `resolved_size` object on every line:

```json
{
  "orderId": "ord_…",
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

Confirm `resolved_size.chest_cm` matches your expectation before placing production volume — size ladders differ materially across systems.

The response also includes the header:

```
X-Printf-Size-System: US
```

## 3 — Handle errors

| Code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | Multi-facility account, no resolvable `size_system` | Add `size_system` to the order or to the affected line |
| `size_system_implicit` | Single-facility account resolved by fallback (warning, becomes error in 2.6) | Add `size_system` before upgrading to 2.6 |

## 4 — Next steps

- [Sizing and fit](./sizing.md) — full size system reference, `fit` values, chest measurement tables
- [Bulk orders and templates](./bulk-orders.md) — multiple lines, saved templates, mixed size systems
