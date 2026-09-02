---
title: Quickstart
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Quickstart

Place your first order in under five minutes.

## Prerequisites

- An API key from your account dashboard
- A design ID (`dsn_…`) from a completed design upload
- Your account ID (`acct_…`)

## Place an order

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
        "fit": "unisex",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }'
```

`size_system` is required at the order level (or per line). Omitting it on a multi-facility account returns `400 size_system_ambiguous`. See [Sizing and fit](/guides/sizing) for the full reference.

## Read the response

```json
{
  "orderId": "ord_abc123",
  "status": "confirmed",
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "quantity": 250,
      "garmentSku": "tee-classic-black",
      "resolved_size": { "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
    }
  ]
}
```

The `resolved_size` object on each line confirms what the facility will cut. The `X-Printf-Size-System` response header echoes the system applied to the order.

## Next steps

- [Bulk orders and templates](/guides/bulk-orders) — place up to 500 lines in a single request
- [Sizing and fit](/guides/sizing) — size system reference, fit options, and the migration guide for saved templates
- [Webhooks](/guides/webhooks) — receive `order.confirmed` and `order.shipped` events, which now include `resolved_size` per line
