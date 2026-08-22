---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-22
owner: platform-docs
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

As of **2.4.0**, every size label must be accompanied by a size system. Bare labels
(`"XL"` with no system) are no longer resolved by guessing the fulfilling facility's
default. This guide explains how to set size system and fit correctly and how to read
the resolved size back from the response.

## Size systems

| Value | Region | US XL chest | JP XL chest |
|---|---|---|---|
| `US` | United States / Canada | 112 cm | — |
| `EU` | Europe | 109 cm | — |
| `JP` | Japan | — | 97 cm |

Ladders differ materially across systems. A `JP` `XL` and a `US` `XL` differ by 15 cm.
Always set the system that matches the end-wearer's measurement convention.

## Setting size system

You can set `size_system` at two levels. The line-level value takes precedence.

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
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "JP",
      "fit": "womens",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Resolution order when `size_system` is absent on a line:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default (if configured)
4. Fulfilling facility's default — **only for single-facility accounts**; multi-facility accounts receive `400 size_system_ambiguous`

## The `fit` field

`fit` is set per line. Accepted values:

| Value | Description |
|---|---|
| `unisex` | Standard unisex chest ladder (default) |
| `womens` | Contoured fit; narrower chest, shorter body |
| `mens` | Straight-cut fit |

Omitting `fit` defaults to `unisex`.

## Reading resolved size

Every line in the response and in webhook payloads now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header shows the system applied to the order
as a whole.

## Error reference

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and no `size_system` was supplied | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account resolved size by facility default | Add `size_system` before 2.6, when this becomes an error |

## SDK support

All five client libraries expose the new fields in 2.4.0:

| SDK | Field access |
|---|---|
| printf-js | `line.size_system`, `line.fit`, `line.resolved_size` |
| printf-py | `line.size_system`, `line.fit`, `line.resolved_size` |
| printf-java | `OrderLine.sizeSystem()`, `OrderLine.fit()`, `OrderLine.resolvedSize()` |
| printf-go | `Line.SizeSystem`, `Line.Fit`, `Line.ResolvedSize` |
| printf-rb | `line.size_system`, `line.fit`, `line.resolved_size` |

