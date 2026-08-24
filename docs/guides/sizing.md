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

As of **order-api 2.4.0**, every size label must be paired with an explicit size system. Bare labels like `XL` are ambiguous across ladders: a JP `XL` chest is 97 cm while a US `XL` chest is 112 cm. Accepting one without knowing the other caused garments to ship in the wrong size.

## Size systems

| `size_system` value | Regions | Example ladder |
|---|---|---|
| `US` | North America | S → M → L → XL → 2XL |
| `EU` | Europe, Australia | 44 → 46 → 48 → 50 |
| `JP` | Japan, East Asia | S → M → L → XL (97 cm) |

## Specifying size system

`size_system` can be set at two levels. The per-line value takes precedence; if omitted, the order-level value applies.

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

To mix systems in a single order — for example, a US domestic line and an EU export line — omit `size_system` at the order level and set it on each line individually.

## Fit

`fit` is a per-line field. Accepted values:

| Value | Description |
|---|---|
| `unisex` | Default cut |
| `mens` | Longer body, wider shoulders |
| `womens` | Contoured cut |

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

Use `chest_cm` for cross-system comparisons in your own tooling.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reflects the effective size system used for the order. Log it — it is your audit trail when a customer queries which ladder was applied.

## Resolution fallback chain

When `size_system` is omitted, the API resolves in this order:

1. Per-line `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default *(routing-dependent)*

Step 4 is only reachable when your account routes to exactly one facility. If your account can route to more than one facility, step 4 is rejected with `400 size_system_ambiguous` — the API will not guess.

## Error and warning reference

| Code | HTTP status | Meaning | When it becomes hard error |
|---|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` is missing and cannot be resolved because the account routes to multiple facilities | Already an error in 2.4 |
| `size_system_implicit` | — (warning) | `size_system` is missing but was resolved via single-facility default | Hard error in **2.6** |

## Saved order templates

Templates are not visible in the API response list, but they are evaluated on every use. If you have saved templates that omit `size_system`, they will produce `size_system_implicit` warnings now and break in 2.6. Open each template in the dashboard and add `size_system` explicitly.

