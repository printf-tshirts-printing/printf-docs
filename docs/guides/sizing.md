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

As of **order-api 2.4.0**, every size label must be accompanied by an explicit size system. A bare `"XL"` is ambiguous — a JP `XL` chest is 97 cm, a US `XL` chest is 112 cm. The API now rejects ambiguous requests rather than guessing.

## Size systems

| Value | Description |
|---|---|
| `US` | United States sizing ladder |
| `EU` | European sizing ladder |
| `JP` | Japanese sizing ladder |

Set `size_system` at the order level as a default for all lines, and override it per line when a single order mixes systems.

## How `size_system` resolves

The API walks this chain and uses the first explicit value it finds:

1. `lines[n].size_system` — line-level (highest priority)
2. `size_system` on the order body — order-level default
3. Account-level default configured in your Printf account settings
4. The fulfilling facility's default — **only if your account routes to exactly one facility**

If resolution reaches step 4 and your account can route to more than one facility, the request is rejected with **`400 size_system_ambiguous`**. If it resolves at step 3 or 4, the response includes a `size_system_implicit` warning. That warning becomes an error in **2.6** — add an explicit `size_system` before then.

## Fit

Each line accepts an optional `fit` field. Valid values depend on your product catalogue, but common values are `unisex`, `mens`, and `womens`. Omitting `fit` defaults to `unisex`.

## The `resolved_size` object

Every line in a successful response and in webhook payloads now includes `resolved_size`, confirming what the API actually applied:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` to cross-check against your own garment spec sheets before committing a large run.

## The `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reflects the size system applied to the order as a whole. Useful for logging and for asserting in integration tests that the system you intended was actually used.

## Full example

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

Response line (abbreviated):

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

## Saved order templates

Templates are not validated until they are used to create an order. A template created before 2.4.0 that omits `size_system` will fail at order-creation time with `400 size_system_ambiguous` (multi-facility accounts) or produce a `size_system_implicit` warning (single-facility accounts). **Review and update all saved templates before 2.6.**

## Error and warning reference

| Code | HTTP status | When it fires |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` is missing and the account routes to more than one facility |
| `size_system_implicit` | — (warning) | `size_system` resolved from account or facility default, not from the payload. Becomes an error in 2.6. |

