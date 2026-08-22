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

As of **order-api 2.4.0**, every size label must carry an explicit size system. Omitting it causes a `400 size_system_ambiguous` error for multi-facility accounts, and a `size_system_implicit` warning (error in 2.6) for single-facility accounts.

## Size systems

| `size_system` | Region | US `XL` chest equivalent |
|---|---|---|
| `US` | United States, Canada | 112 cm |
| `EU` | Europe | 112 cm (different ladder at smaller sizes) |
| `JP` | Japan | 97 cm |

Ladders differ materially. A JP `XL` and a US `XL` are **different garments**. Always declare the system.

## Declaring the system

You can set `size_system` at two scopes. The most specific value wins.

| Scope | Field | Applies to |
|---|---|---|
| Order | `size_system` | All lines that omit their own `size_system` |
| Line | `size_system` | That line only |

### Order-level declaration (all lines share a system)

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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

### Line-level override (mixed systems in one order)

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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 100
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 50
    }
  ]
}
```

The two `XL` lines above resolve to different physical garments (112 cm vs 97 cm chest).

## The `fit` field

Each line accepts an optional `fit` value: `unisex` (default), `womens`, or `mens`. Fit affects the cut ladder independently of the size system.

## Reading `resolved_size` in responses

Every response line now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Verify `chest_cm` against your garment spec before confirming large print runs. The response header `X-Printf-Size-System` echoes the system resolved for the request as a whole.

## Webhook payloads

`resolved_size` is included on every line item in webhook payloads, using the same shape as the API response. No webhook schema changes are required to receive it, but you should start reading it.

## Error reference

| Code | HTTP | Meaning | Remedy |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and no `size_system` was provided at any level | Add `size_system` to the order or each line |
| `size_system_implicit` | warning (400 in 2.6) | Single-facility account omitted `size_system`; resolved from facility default | Add `size_system` before upgrading to 2.6 |

