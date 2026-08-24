---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: dx
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

As of **2.4.0**, every size label must be paired with a size system. The API no longer guesses.
A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm — silent misresolution shipped wrong-sized garments.

## Size systems

| Value | Region | XL chest |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 104 cm |
| `JP` | Japan | 97 cm |

## Setting the size system

You can set `size_system` at two levels. The line-level value wins when both are present.

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

**Resolution order when `size_system` is absent from a line:**

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default _(only for single-facility accounts — see below)_

## Fit

Each line accepts an optional `fit` field. Supported values depend on the garment SKU; consult the
catalogue endpoint. Common values: `unisex`, `fitted`, `relaxed`.

## `resolved_size` in responses

Every response line and webhook payload now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use this field to confirm the system that was actually applied — especially when relying on order-level or account defaults.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reports the size system used to resolve all lines in the
request. This is useful for logging and debugging, particularly when `size_system` was inherited
rather than set explicitly.

## Multi-facility accounts

If your account can route to more than one facility and you omit `size_system` from both the order
body and every line, the API returns:

```
400 size_system_ambiguous
```

This replaces the previous behaviour of resolving by guess. Set an explicit `size_system` on the
order or per line to resolve this error.

## Single-facility accounts

If your account routes to exactly one facility and you omit `size_system`, the API resolves using
that facility's default and attaches a warning:

```
size_system_implicit
```

This warning becomes an error in **2.6**. Set `size_system` now to avoid a breaking change at upgrade.

