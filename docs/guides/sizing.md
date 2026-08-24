---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-rb
  - printf-go
---

# Sizing and fit

As of **order-api 2.4.0**, every order line carries an explicit size system. Bare size labels are no longer resolved by guessing.

## Size systems

| Value | Region | XL chest (cm) |
|-------|--------|---------------|
| `US`  | United States / Canada | 112 |
| `EU`  | Europe | 104 |
| `JP`  | Japan  | 97  |

Ladders differ materially. A JP `XL` is 15 cm narrower in the chest than a US `XL`. Always pass `size_system` explicitly to avoid fulfillment errors.

## Where to set `size_system`

`size_system` can appear at two levels of the request:

| Level | Field | Scope |
|-------|-------|-------|
| Order root | `size_system` | Default for all lines that omit their own `size_system` |
| Line | `size_system` | Overrides the order-level value for that line |

The resolution chain is: **line → order → account default → fulfilling facility default**.

Accounts routable to more than one facility cannot fall through to the facility default — the request is rejected with `400 size_system_ambiguous`. Pass `size_system` at the order or line level to resolve this.

## The `fit` field

Each line accepts a `fit` value:

| Value | Description |
|-------|-------------|
| `unisex` | Unisex cut (default when omitted on older requests; set explicitly from 2.4.0) |
| `mens` | Mens cut |
| `womens` | Womens cut |

## Request example

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

## Response: `resolved_size`

Every line in the response now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header contains the system actually applied to the order. Use this to confirm which system was resolved when `size_system` was not set at the line level.

## Warning: `size_system_implicit`

If `size_system` is absent and your account has a single fulfilling facility, the request succeeds with a `size_system_implicit` warning in the response body. **This warning becomes an error in 2.6.** Set `size_system` explicitly on every order or line before the 2.6 release to avoid breakage.

## Saved order templates

Templates do not appear in the API response for `GET /v2/orders`, but they are affected. Any template created before 2.4.0 that does not include `size_system` will produce a `size_system_implicit` warning on fulfillment. Review and update all saved templates before 2.6.

