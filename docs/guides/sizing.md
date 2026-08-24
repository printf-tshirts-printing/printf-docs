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
  - printf-rb
  - printf-go
---

# Sizing and fit

As of **order-api 2.4.0**, every size label must be accompanied by the system it comes from. This page explains how `size_system` and `fit` work, how `resolved_size` lets you verify the ladder that was applied, and what you need to change if you were relying on implicit resolution.

## Size systems

Printf supports three size systems:

| Value | Region | US `XL` equivalent (chest) |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | varies by label — check `resolved_size.chest_cm` |
| `JP` | Japan | 97 cm for `XL` |

A JP `XL` is 15 cm narrower than a US `XL`. Omitting `size_system` and relying on a facility default will produce the wrong garment for cross-border orders.

## Specifying `size_system`

`size_system` can be set at two levels and resolves in this order:

1. **Line level** — `lines[].size_system` — highest precedence.
2. **Order level** — top-level `size_system` — applies to every line that omits it.
3. Account default — only used for single-facility accounts; emits `size_system_implicit` warning.
4. Facility default — only used when account routing is unambiguous; emits `size_system_implicit` warning.

Multi-facility accounts that reach step 3 or 4 receive a `400 size_system_ambiguous` error instead.

### Order-level `size_system`

Set `size_system` once on the order and every line inherits it:

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
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### Per-line `size_system`

Override the order-level system on individual lines when an order mixes regions:

```json
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
    "size": "XL",
    "size_system": "JP",
    "fit": "unisex",
    "quantity": 100,
    "garmentSku": "tee-classic-black"
  }
]
```

The two lines above resolve to different physical garments even though both carry the label `XL`.

## `fit`

`fit` is an optional per-line qualifier. Accepted values depend on the `garmentSku`; common values are `unisex`, `womens`, and `slim`. If omitted, the SKU's default fit is used.

## Verifying resolution with `resolved_size`

Every line in the response and in webhook payloads now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Always log or assert `resolved_size.chest_cm` in your integration tests. A mismatch here means the wrong ladder was applied and the garment will be cut to the wrong measurement.

## `X-Printf-Size-System` response header

The response includes a top-level header reflecting the size system applied to the order:

```
X-Printf-Size-System: US
```

Useful for logging and audit without parsing the full response body.

## Error and warning reference

| Code | HTTP status | When it fires | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` provided | Add `size_system` to the order or per line |
| `size_system_implicit` | — (warning in 2.x) | Single-facility account, `size_system` inferred from facility default | Add `size_system` explicitly before 2.6 |

`size_system_implicit` **becomes a hard `400` error in 2.6.** Add `size_system` now to avoid a hard cutover.

