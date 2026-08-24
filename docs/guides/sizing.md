---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of **order-api 2.4.0**, every size label must be paired with an explicit size system. Bare labels without a system are still accepted but produce a warning that becomes an error in 2.6. If you are on an earlier version of a client library, upgrade before sending orders that cross fulfillment facilities.

## Size systems

| `size_system` value | Region | Example: label `XL` | Chest (cm) |
|---|---|---|---|
| `US` | United States / Canada | XL | 112 |
| `EU` | Europe | XL | 107 |
| `JP` | Japan | XL | 97 |

Size ladders differ materially between systems. A JP `XL` is 97 cm; a US `XL` is 112 cm. Sending the wrong system produces a garment in the wrong size — no error, just a misfulfilled order.

## Where to set `size_system`

You can set `size_system` at two levels. The more specific value wins.

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` | Default for all lines in the order |
| Line | `size_system` | Overrides the order-level default for that line only |

If neither level is set, the API falls back to the fulfilling facility's default — **only** when your account routes to a single facility. Multi-facility accounts receive `400 size_system_ambiguous` immediately.

## Fit

Each line accepts a `fit` field. Valid values depend on the garment; common values are `unisex`, `mens`, and `womens`. Fit is reflected back in `resolved_size` on the response.

## `resolved_size` on responses

Every line in the response now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The same object appears in webhook payloads. Use `chest_cm` to verify fulfillment intent before downstream processing.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` contains the size system that was used to resolve all lines in the order. When every line shares a system this is a single value (e.g. `US`). Log this header to make post-order debugging faster.

## Example request

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

The line-level `size_system` here matches the order-level value. You only need both when mixing systems within a single order.

## Error and warning codes

| Code | HTTP status | When it fires |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` on the request and the account can route to more than one facility |
| `size_system_implicit` | 200 (warning in body) | No `size_system` on the request; resolved from the single facility's default. **Becomes a 400 error in 2.6.** |

Handle `size_system_implicit` now. Do not wait for 2.6.
