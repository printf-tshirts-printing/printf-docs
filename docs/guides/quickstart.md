---
title: Quickstart
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
---

# Quickstart

This guide creates your first order using the `printf-js` SDK.

> **printf-js SDK note (2.4.0):** `printf-js` does not yet model `size_system`, `fit`, or `resolved_size`. Pass these fields using the `extraFields` escape hatch shown below until a new SDK release ships.

## Prerequisites

- A Printf account (`accountId`)
- An API key
- Node 18 or later

## Install

```bash
npm install printf-js
```

## Create an order

As of order-api **2.4.0**, `size_system` and `fit` are required on every order. Omitting them produces a `size_system_implicit` deprecation warning on single-facility accounts and a `400 size_system_ambiguous` error on multi-facility accounts.

```js
import { PrintfClient } from 'printf-js';

const client = new PrintfClient({ apiKey: process.env.PRINTF_API_KEY });

const order = await client.orders.create({
  accountId: 'acct_stackfest',
  facilityId: 'fac-atx',
  destination: {
    name: 'StackFest Ops',
    line1: '410 Congress Ave',
    city: 'Austin',
    region: 'TX',
    postalCode: '78701',
    countryCode: 'US',
  },
  // size_system and fit are not yet typed in printf-js; pass via extraFields
  extraFields: {
    size_system: 'US',
  },
  lines: [
    {
      designId: 'dsn_7fa91c',
      size: 'XL',
      fit: 'unisex',
      quantity: 250,
      garmentSku: 'tee-classic-black',
      // line-level size_system overrides order-level when systems are mixed
      extraFields: {
        size_system: 'US',
      },
    },
  ],
});

console.log(order.lines[0].resolved_size);
// { label: 'XL', system: 'US', fit: 'unisex', chest_cm: 112 }
```

## Inspect resolved_size

Every line in the response includes `resolved_size`. Use `chest_cm` for any downstream logic — do not compare `label` across systems.

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

## Response header

The response includes `X-Printf-Size-System` set to the system applied to the order. Useful for logging and debugging.

## Next steps

- [Sizing and fit](./sizing.md) — full size ladder tables and multi-system orders
- [Bulk orders and templates](./bulk-orders.md) — update saved templates to include `size_system`
- [Webhooks](./webhooks.md) — `resolved_size` is also present in webhook payloads

