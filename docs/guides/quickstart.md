---
title: Quickstart
section: guides
last_reviewed: 2026-08-21
owner: dx
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
---

# Quickstart

This guide walks you through placing your first order with the Printf API using the `printf-js` SDK.

> **SDK note — size system support:** `printf-js` v2.3.4 does not yet model `size_system`, `fit`, or `resolved_size`. Pass these fields as plain objects until an updated SDK ships. The raw HTTP examples below work today.

## Prerequisites

- A Printf account and API key
- Node.js 18+
- `printf-js` installed: `npm install printf-js`

## Place an order

### HTTP

```http
POST /v2/orders
Content-Type: application/json
Authorization: Bearer <api-key>
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

A successful `201` response includes `resolved_size` on every line:

```json
{
  "orderId": "ord_abc123",
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

The response also includes the `X-Printf-Size-System: US` header, confirming which size system was applied to the order.

### Check what you resolved

Inspect `resolved_size.chest_cm` to verify the garment dimensions before fulfillment. A US `XL` resolves to 112 cm chest; a JP `XL` resolves to 97 cm — a 15 cm difference that cannot be corrected after production starts.

### printf-js SDK (pass-through until SDK update ships)

```js
import { PrintfClient } from 'printf-js';

const client = new PrintfClient({ apiKey: process.env.PRINTF_API_KEY });

const order = await client.orders.create({
  accountId: 'acct_stackfest',
  size_system: 'US',           // add this — not yet typed in SDK v2.3.4
  facilityId: 'fac-atx',
  destination: {
    name: 'StackFest Ops',
    line1: '410 Congress Ave',
    city: 'Austin',
    region: 'TX',
    postalCode: '78701',
    countryCode: 'US',
  },
  lines: [
    {
      designId: 'dsn_7fa91c',
      size: 'XL',
      size_system: 'US',       // add this
      fit: 'unisex',           // add this
      quantity: 250,
      garmentSku: 'tee-classic-black',
    },
  ],
});

// resolved_size is on each line in the raw response object
console.log(order.lines[0].resolved_size);
// { label: 'XL', system: 'US', fit: 'unisex', chest_cm: 112 }
```

## Next steps

- [Sizing and fit](sizing.md) — size system resolution rules, fit options, and ladder tables
- [Bulk orders and templates](bulk-orders.md) — updating saved templates for `size_system`
- [Webhooks](webhooks.md) — `resolved_size` in webhook payloads

