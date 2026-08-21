---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-21
owner: fulfillment
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
---

# Bulk orders and templates

## Migration required for order-api 2.4.0 {#migration-240}

> **If you use saved order templates, read this section first.** Templates are not visible in the API — they are stored server-side and replayed on each bulk run. A template that does not include `size_system` will start failing or producing warnings from order-api 2.4.0 onward.

See the [full migration note](#) for complete before/after payloads and error codes.

---

## Bulk order creation

Bulk orders use the same `POST /v2/orders` endpoint called in a loop or batch. As of 2.4.0, every order in the batch must carry `size_system` at order level, line level, or both.

### Before (order-api ≤ 2.3.x) — will fail or warn from 2.4.0

```json
{
  "accountId": "acct_stackfest",
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
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

**What happens now:**
- Multi-facility account → `400 size_system_ambiguous` — order rejected.
- Single-facility account → accepted with `size_system_implicit` warning — **becomes `400` in 2.6**.

### After (order-api 2.4.0+) — correct form

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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

The response now includes `resolved_size` on each line:

```json
{
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

## Updating saved templates

Saved order templates are stored server-side and are not returned by the API. To update them:

1. Retrieve your template definitions from wherever you store them (internal config, IaC, spreadsheets).
2. Add `size_system` at the order level and/or on each `lines[]` entry.
3. Add `fit` to each line.
4. Re-save or re-register the template.
5. Run a test order and verify `resolved_size.chest_cm` in the response matches your expectation.

If you are unsure which templates are affected, search your codebase for `POST /v2/orders` call sites and any serialized JSON blobs containing `garmentSku`.

## printf-js bulk example (2.4.0+)

> `printf-js` v2.3.4 does not yet type `size_system`, `fit`, or `resolved_size`. Pass them as plain properties — they round-trip correctly.

```js
import { PrintfClient } from 'printf-js';

const client = new PrintfClient({ apiKey: process.env.PRINTF_API_KEY });

const orders = await Promise.all(batch.map(item =>
  client.orders.create({
    accountId: item.accountId,
    size_system: 'US',
    destination: item.destination,
    lines: item.lines.map(line => ({
      designId: line.designId,
      size: line.size,
      size_system: 'US',
      fit: line.fit ?? 'unisex',
      quantity: line.quantity,
      garmentSku: line.garmentSku,
    })),
  })
));
```

## printf-py bulk example (2.4.0+)

> `printf-py` v2.3.2 does not yet type `size_system`, `fit`, or `resolved_size`. Pass them as plain dict keys.

```python
from printf import PrintfClient
import os

client = PrintfClient(api_key=os.environ["PRINTF_API_KEY"])

for item in batch:
    order = client.orders.create({
        "accountId": item["accountId"],
        "size_system": "US",
        "destination": item["destination"],
        "lines": [
            {
                "designId": line["designId"],
                "size": line["size"],
                "size_system": "US",
                "fit": line.get("fit", "unisex"),
                "quantity": line["quantity"],
                "garmentSku": line["garmentSku"],
            }
            for line in item["lines"]
        ],
    })
    for line in order["lines"]:
        assert line["resolved_size"]["system"] == "US"
```

