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
  - printf-rb
  - printf-go
---

# Quickstart

Place your first order in under five minutes. This guide uses **order-api 2.4.0**. If you are on an earlier version, `size_system` and `fit` are not available — upgrade before following these steps.

## Prerequisites

- An account ID (`accountId`) — find it in the dashboard under **Settings → Account**.
- A design ID (`designId`) — create a design in the dashboard or via `POST /v2/designs`.
- A garment SKU (`garmentSku`) — browse available SKUs in the dashboard under **Catalogue**.
- An API key — **Settings → API keys → New key**.

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
        "garmentSku": "tee-classic-black",
        "size": "XL",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 250
      }
    ]
  }'
```

Replace `acct_stackfest`, `dsn_7fa91c`, and `tee-classic-black` with your own values.

## Read the response

A successful `201` response includes `resolved_size` on every line:

```json
{
  "orderId": "ord_abc123",
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

Check `resolved_size.system` to confirm the system that was used. Check `resolved_size.chest_cm` to confirm the physical measurement — this is the fastest way to catch a mismatched size system before garments are cut.

The response also includes the header:

```
X-Printf-Size-System: US
```

## Required fields — quick reference

| Level | Required fields |
|---|---|
| Order | `accountId`, `destination` |
| Each line | `designId`, `garmentSku`, `quantity` |
| Strongly recommended | `size_system` on order and per line, `fit` per line |

`size_system` is not technically required on every request today — a single-facility account will fall back to the facility default — but omitting it triggers a `size_system_implicit` warning and will become a `400` error in **2.6.0**. Add it now.

## SDK examples

### printf-js

```js
import { PrintfClient } from '@printf/printf-js';

const client = new PrintfClient({ apiKey: process.env.PRINTF_API_KEY });

const order = await client.orders.create({
  accountId: 'acct_stackfest',
  size_system: 'US',
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
      garmentSku: 'tee-classic-black',
      size: 'XL',
      size_system: 'US',
      fit: 'unisex',
      quantity: 250,
    },
  ],
});

console.log(order.lines[0].resolved_size); // { label: 'XL', system: 'US', fit: 'unisex', chest_cm: 112 }
```

### printf-py

```python
from printf import PrintfClient

client = PrintfClient(api_key=os.environ["PRINTF_API_KEY"])

order = client.orders.create(
    account_id="acct_stackfest",
    size_system="US",
    facility_id="fac-atx",
    destination={
        "name": "StackFest Ops",
        "line1": "410 Congress Ave",
        "city": "Austin",
        "region": "TX",
        "postalCode": "78701",
        "countryCode": "US",
    },
    lines=[
        {
            "designId": "dsn_7fa91c",
            "garmentSku": "tee-classic-black",
            "size": "XL",
            "size_system": "US",
            "fit": "unisex",
            "quantity": 250,
        }
    ],
)

print(order.lines[0].resolved_size)  # ResolvedSize(label='XL', system='US', fit='unisex', chest_cm=112)
```

## Next steps

- [Sizing and fit](./sizing.md) — full reference for `size_system`, `fit`, and `resolved_size`
- [Bulk orders and templates](./bulk-orders.md) — submit large orders and manage reusable templates

