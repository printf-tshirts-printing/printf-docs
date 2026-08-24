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
  - printf-go
  - printf-rb
---

# Quickstart

This guide gets you from zero to a placed order in five minutes. It reflects order-api **2.4.0**. If you are upgrading from 2.3 or earlier, see the [migration note](#upgrading-from-23).

## Prerequisites

- A Printf account with an `accountId`
- An API key (set as `Authorization: Bearer <key>`)
- A design already uploaded (`designId`)

## Place your first order

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

### What changed in 2.4.0

`size_system` is now a required concern. You must supply it at order level, per line, or both. Omitting it on a multi-facility account returns `400 size_system_ambiguous`. On a single-facility account you get a warning today and a hard error in 2.6.

Always include `size_system` — it is two characters and it prevents wrong-size fulfilment.

## Understand the response

```json
{
  "orderId": "ord_a1b2c3",
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

The `resolved_size` object on each line confirms what the API understood. Check `chest_cm` — if it does not match your expectation, the wrong `size_system` was applied.

The response also includes the `X-Printf-Size-System` header echoing the system applied at order level.

## Required fields

Every request to `POST /v2/orders` must include:

| Field | Notes |
|---|---|
| `accountId` | Your account identifier |
| `destination` | Full address object including `countryCode` |
| `lines[].designId` | Upload your design first |
| `lines[].garmentSku` | Must match a live SKU in the catalog |
| `lines[].quantity` | Integer, minimum 1 |

Missing any of these returns `400 missing_required_field`.

## Next steps

- [Sizing and fit](./sizing.md) — full `size_system` reference and chest measurements by system
- [Bulk orders and templates](./bulk-orders.md) — multi-line orders and saved template audit for 2.4.0
- [Webhooks](./webhooks.md) — `resolved_size` now appears in webhook payloads too

## Upgrading from 2.3

See the [full migration note](../migration-notes/2.4.0.md). The short version is below.

**Before (2.3):**
```json
{
  "accountId": "acct_stackfest",
  "destination": { "name": "StackFest Ops", "line1": "410 Congress Ave", "city": "Austin", "region": "TX", "postalCode": "78701", "countryCode": "US" },
  "lines": [{ "designId": "dsn_7fa91c", "size": "XL", "quantity": 250, "garmentSku": "tee-classic-black" }]
}
```

**After (2.4):**
```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "facilityId": "fac-atx",
  "destination": { "name": "StackFest Ops", "line1": "410 Congress Ave", "city": "Austin", "region": "TX", "postalCode": "78701", "countryCode": "US" },
  "lines": [{ "designId": "dsn_7fa91c", "size": "XL", "size_system": "US", "fit": "unisex", "quantity": 250, "garmentSku": "tee-classic-black" }]
}
```

