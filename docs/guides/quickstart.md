---
title: Quickstart
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js]
---

# Quickstart

Place your first order with `POST /v2/orders`.

Since order-api 2.4.0, every order must say which size system its sizes use. This is a breaking change. If your account can route to more than one facility, a request with a bare `size` fails with `400 size_system_ambiguous`.

## 1. Build the request

Send `size_system` (`US`, `EU` or `JP`) on the order. `accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required.

```json
POST /v2/orders
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

| Field | Where | Purpose |
|---|---|---|
| `size_system` | Order, and optionally each line | `US`, `EU` or `JP`. A line value overrides the order value. |
| `fit` | Each line | The fit of the garment on that line |

## 2. Check the resolved size

Each response line carries `resolved_size`. Confirm it matches what you meant to order.

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Responses also carry an `X-Printf-Size-System` header. Webhook payloads carry `resolved_size` too.

## 3. Handle size errors

| Code | Kind | What to do |
|---|---|---|
| `size_system_ambiguous` | Error, `400` | Send `size_system` on the order or the line |
| `size_system_implicit` | Warning | Send `size_system`. This becomes an error in 2.6. |

:::warning
A JP `XL` is 97 cm and a US `XL` is 112 cm. Do not send a bare `size` and rely on a default. Always send `size_system`.
:::

## Next

- [Sizing and fit](/guides/sizing) covers how a bare size resolves and how to migrate saved templates.
- [Bulk orders and templates](/guides/bulk-orders) covers mixed-system orders.

