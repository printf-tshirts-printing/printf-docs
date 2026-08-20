---
title: JavaScript and TypeScript
section: sdks
last_reviewed: 2026-05-22
owner: devex
covers_sdks: [printf-js]
covers_endpoints: [POST /v2/orders, GET /v2/orders/{id}]
---

# JavaScript and TypeScript

```bash
npm install @printf/sdk
```

```ts
import { PrintfClient } from "@printf/sdk";

const printf = new PrintfClient({ apiKey: process.env.PRINTF_API_KEY! });

const order = await printf.createOrder({
  accountId: "acct_stackfest",
  destination: {
    name: "StackFest Ops",
    line1: "410 Congress Ave",
    city: "Austin",
    region: "TX",
    postalCode: "78701",
    countryCode: "US",
  },
  lines: [
    { designId: "dsn_7fa91c", size: "L", quantity: 250, garmentSku: "tee-classic-black" },
    { designId: "dsn_7fa91c", size: "XL", quantity: 180, garmentSku: "tee-classic-black" },
  ],
});

console.log(order.id, order.facilityId);
```

## Errors

```ts
import { PrintfApiError } from "@printf/sdk";

try {
  await printf.createOrder(request);
} catch (error) {
  if (error instanceof PrintfApiError && error.code === "size_unavailable") {
    // handle
  }
  throw error;
}
```

## Sandbox

```ts
new PrintfClient({ apiKey, baseUrl: "https://api.sandbox.printf.dev" });
```

Sandbox accepts orders, returns realistic responses, and prints nothing.
