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

Place your first order with the Printf order API in under five minutes.

## Prerequisites

- An API key — generate one at **Account → API keys** in the Printf dashboard.
- Your `accountId` — shown on the **Account → Overview** page.
- Your `facilityId` — listed under **Account → Facilities**.

## Place an order

The minimum viable request. Every field shown is required; the request is rejected if any is absent.

> **order-api 2.4.0+** — `size_system` is required if your account routes to more than one facility. Omitting it returns `400 size_system_ambiguous`. Add it to every order.

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

## Read the response

A successful response is `201 Created`. Each line now includes `resolved_size`:

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

Confirm `resolved_size.chest_cm` matches your design before proceeding. A US `XL` is 112 cm; a JP `XL` is 97 cm — they are not interchangeable.

The response also includes the header:

```
X-Printf-Size-System: US
```

## Next steps

- [Sizing and fit](docs/guides/sizing.md) — full size system reference, fit values, and the `resolved_size` schema.
- [Bulk orders and templates](docs/guides/bulk-orders.md) — multi-line orders and updating saved templates.

