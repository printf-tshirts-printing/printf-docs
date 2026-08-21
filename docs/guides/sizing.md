---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
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

As of **order-api 2.4.0**, every order can carry an explicit `size_system` and every line item can carry a `fit`. Responses and webhook payloads include a `resolved_size` object so you can confirm exactly what was resolved.

## Size systems

| System | Token | Example XL chest |
|--------|-------|------------------|
| United States | `US` | 112 cm |
| European | `EU` | 116 cm |
| Japanese | `JP` | 97 cm |

Ladders differ materially. A JP `XL` and a US `XL` are not interchangeable. **Always supply `size_system` explicitly.**

## Resolution order

When `size_system` is present at multiple levels, the most specific wins:

1. Per-line `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default *(only reached if the account routes to exactly one facility)*

If your account can route to more than one facility and you reach step 4, the API returns `400 size_system_ambiguous`. There is no silent guess.

## Specifying size system and fit

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
      "quantity": 250
    }
  ]
}
```

The order-level `size_system: "US"` acts as a default for all lines. The per-line `size_system` overrides it for that line only.

## The `resolved_size` response object

Every line in the response body and in webhook payloads now includes:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

| Field | Type | Description |
|-------|------|-------------|
| `label` | string | The size token you submitted |
| `system` | string | The system that was used to resolve the label |
| `fit` | string | The fit that was resolved (`unisex`, `mens`, `womens`) |
| `chest_cm` | number | Chest measurement in centimetres for this resolved size |

The `X-Printf-Size-System` response header reflects the system resolved for the order as a whole.

## `fit` values

| Token | Description |
|-------|-------------|
| `unisex` | Default when `fit` is omitted |
| `mens` | Men's cut |
| `womens` | Women's cut |

## Per-line override example

You can mix systems within one order using per-line overrides:

```json
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
      "garmentSku": "tee-classic-white",
      "size": "XL",
      "size_system": "JP",
      "fit": "womens",
      "quantity": 50
    }
  ]
}
```

The second line resolves to 97 cm chest regardless of the order-level `US` default.

## Error and warning codes

| Code | HTTP status | Meaning | Action |
|------|-------------|---------|--------|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and no `size_system` was resolved | Add `size_system` to the order or a line |
| `size_system_implicit` | — (warning) | Account routes to one facility; system was inferred from facility default | Add explicit `size_system` before 2.6.0 |

