---
title: Quickstart
section: guides
last_reviewed: 2026-08-28
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Quickstart

Place your first order in five minutes. This guide uses `POST /v2/orders` directly; client library examples follow.

## Before you start

You need:

- An `accountId` (from the dashboard, format `acct_…`)
- An API key with `orders:write` scope
- A `designId` for the artwork you want printed
- A `garmentSku` — see the [catalogue](/guides/catalogue)

## Place an order

As of **Orders API 2.4.0**, every order must include `size_system`. Accepted values: `US`, `EU`, `JP`.

```json
POST /v2/orders
Authorization: Bearer <api_key>
Content-Type: application/json

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

### Response

```json
{
  "orderId": "ord_…",
  "status": "accepted",
  "lines": [
    {
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

The `X-Printf-Size-System` response header echoes the resolved system for the order.

Check `resolved_size` on each line to confirm the system that was applied. A JP `XL` is 97 cm; a US `XL` is 112 cm — wrong system means wrong garment.

## Common errors

| Code | HTTP status | Cause | Fix |
|------|-------------|-------|-----|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and `size_system` was omitted | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account; `size_system` resolved from facility default | Add explicit `size_system` before 2.6 |

## Client libraries

### JavaScript (printf-js)

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
      size: 'XL',
      size_system: 'US',
      fit: 'unisex',
      quantity: 250,
      garmentSku: 'tee-classic-black',
    },
  ],
});

console.log(order.lines[0].resolved_size); // { label: 'XL', system: 'US', fit: 'unisex', chest_cm: 112 }
```

### Python (printf-py)

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
            "size": "XL",
            "size_system": "US",
            "fit": "unisex",
            "quantity": 250,
            "garmentSku": "tee-classic-black",
        }
    ],
)

print(order.lines[0].resolved_size)  # ResolvedSize(label='XL', system='US', fit='unisex', chest_cm=112)
```

### Java (printf-java)

```java
PrintfClient client = new PrintfClient(System.getenv("PRINTF_API_KEY"));

Order order = client.orders().create(
    OrderCreateRequest.builder()
        .accountId("acct_stackfest")
        .sizeSystem(SizeSystem.US)
        .facilityId("fac-atx")
        .destination(Destination.builder()
            .name("StackFest Ops")
            .line1("410 Congress Ave")
            .city("Austin")
            .region("TX")
            .postalCode("78701")
            .countryCode("US")
            .build())
        .lines(List.of(
            OrderLine.builder()
                .designId("dsn_7fa91c")
                .size("XL")
                .sizeSystem(SizeSystem.US)
                .fit(Fit.UNISEX)
                .quantity(250)
                .garmentSku("tee-classic-black")
                .build()
        ))
        .build()
);

System.out.println(order.getLines().get(0).getResolvedSize().getChestCm()); // 112
```

## Next steps

- [Sizing and fit](/guides/sizing) — full ladder tables, `fit` values, and migration guide for existing templates
- [Bulk orders and templates](/guides/bulk-orders) — orders with many lines and saved templates
- [Webhooks](/guides/webhooks) — `resolved_size` in event payloads

