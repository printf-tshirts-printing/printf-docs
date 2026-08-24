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

As of **2.4.0**, every size label must be accompanied by a size system. A bare `"XL"` is ambiguous — the same label maps to meaningfully different chest measurements depending on the market the garment is cut for.

## Size systems

Three systems are supported:

| `size_system` | Coverage | Notes |
|--------------|----------|-------|
| `US` | North America | Default for most US-based facilities |
| `EU` | Europe | Follows EN 13402 conventions |
| `JP` | Japan | JIS standard; runs narrower than US and EU |

You can set `size_system` at two levels:

- **Order root** — applies to every line that does not set its own.
- **Line item** — overrides the order-root value for that line only.

The lookup order when `size_system` is absent on a line is: line → order → account default → facility default. The facility-default step depends on routing. If your account can route to more than one facility and no system is supplied anywhere in the payload, the request is rejected with `400 size_system_ambiguous`. There is no guess.

## The `fit` field

Each line item accepts a `fit` value:

| `fit` | Description |
|-------|-------------|
| `unisex` | Straight cut (default if omitted) |
| `womens` | Contoured cut; size ladder differs from unisex |
| `mens` | Tapered cut |

`fit` is not inherited from the order root. Set it explicitly on every line where it matters.

## Resolved size in responses

Every line in a response body now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header contains the system that was applied at the order level. Log this header when debugging size discrepancies.

## Example: order with explicit size system

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

The response line for this order will include:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

## Example: per-line override for a mixed-market order

If you are shipping US and EU sizes in the same order, set `size_system` at the line level and omit it (or use your predominant system) at the order root:

```json
{
  "accountId": "acct_stackfest",
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
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "EU",
      "fit": "womens",
      "quantity": 50,
      "garmentSku": "tee-classic-white"
    }
  ]
}
```

Note that a US `XL` (112 cm chest) and an EU `XL` (106 cm chest) are **different garments**. Mixing systems without per-line overrides will silently produce the wrong size for part of your run.

## Size ladder reference

| Label | US chest (cm) | EU chest (cm) | JP chest (cm) |
|-------|--------------|--------------|------------------|
| S | 92 | 88 | 80 |
| M | 99 | 96 | 88 |
| L | 106 | 101 | 92 |
| XL | 112 | 106 | 97 |
| 2XL | 119 | 111 | 102 |

## Error reference

| Code | HTTP status | Meaning |
|------|-------------|--------|
| `size_system_ambiguous` | 400 | Multi-facility account sent a bare size label with no `size_system` anywhere in the payload. |
| `size_system_implicit` | — (warning) | Single-facility account sent a bare size label. The system was inferred from the facility default. Becomes a `400` error in **2.6**. |

