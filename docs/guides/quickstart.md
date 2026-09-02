---
title: Quickstart
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Quickstart

This guide gets you from zero to your first confirmed order in under ten minutes.

## Prerequisites

- A Printf account and API key. Sign up at [printf.dev](https://printf.dev).
- Your account ID (`acct_…`) from the dashboard.
- A facility ID (`fac-…`) — find it under **Settings → Facilities**.

## Your first order

As of Orders API 2.4.0, every order requires an explicit `size_system`. The values are `US`, `EU`, and `JP`.

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
        "quantity": 1,
        "garmentSku": "tee-classic-black"
      }
    ]
  }'
```

A `200` response means the order is confirmed. Each line carries `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response header `X-Printf-Size-System` confirms the system applied to the order.

## What each field does

| Field | Required | Notes |
|---|---|---|
| `accountId` | Yes | Your account identifier. |
| `size_system` | Yes (from 2.4.0) | `US`, `EU`, or `JP`. Set at order level, line level, or both. Line takes precedence. |
| `facilityId` | Yes | The fulfilling facility. |
| `destination` | Yes | Shipping address. |
| `lines[].designId` | Yes | The design to print. |
| `lines[].size` | Yes | Size label within the chosen system. |
| `lines[].fit` | No | Fit variant. Defaults to SKU default if omitted. |
| `lines[].quantity` | Yes | Units to produce. |
| `lines[].garmentSku` | Yes | The garment to print on. |

## Size systems at a glance

| Value | Standard | XL chest |
|---|---|---|
| `US` | US/CA unisex | 112 cm |
| `EU` | European | 104 cm |
| `JP` | Japanese Industrial Standard | 97 cm |

The systems are not interchangeable. A JP `XL` is 15 cm narrower than a US `XL`. Always confirm which system your garments are measured in before placing an order.

## If you receive a 400

`400 size_system_ambiguous` means your account can route to more than one facility and no `size_system` was present on the order or line. Add `size_system` to the request.

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in the response rather than a 400. This becomes a 400 in 2.6 — add `size_system` before then.

## Next steps

- [Sizing and fit](/guides/sizing) — full size ladder reference and migration guide.
- [Bulk orders and templates](/guides/bulk-orders) — how to update saved templates before 2.6.
- [Webhooks](/guides/webhooks) — `resolved_size` now appears in `order.created` and `order.updated` payloads.
