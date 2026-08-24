---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of **order-api 2.4.0**, every size label must be anchored to a size system. Bare labels are deprecated for single-facility accounts and rejected outright for multi-facility accounts.

## Size systems

| `size_system` value | Region | US XL chest equivalent |
|---|---|---|
| `US` | United States | 112 cm |
| `EU` | Europe | 110 cm |
| `JP` | Japan | 97 cm |

A JP `XL` and a US `XL` differ by 15 cm in the chest. Omitting `size_system` and relying on a facility default is the single most common cause of wrong-size fulfilment.

## Specifying the size system

You can set `size_system` at two levels. The per-line value wins when both are present.

### Order level

Applies to every line that does not set its own `size_system`.

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

### Per-line override

Use this when a single order mixes garments from different regional size ladders.

```json
"lines": [
  {
    "designId": "dsn_7fa91c",
    "size": "XL",
    "size_system": "JP",
    "fit": "unisex",
    "quantity": 250,
    "garmentSku": "tee-classic-black"
  }
]
```

## Fit

The `fit` field is accepted per line. Current values: `unisex`, `womens`, `mens`. Fit affects the size ladder mapping — a `womens` `M` does not map to the same chest measurement as a `unisex` `M`.

## resolved_size in responses

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The same object appears in webhook payloads for `order.fulfilled` and `order.line_updated` events. Use `resolved_size.chest_cm` to verify that the garment dispatched matches what your customer ordered — do not re-derive it from the label alone.

## X-Printf-Size-System response header

Every `POST /v2/orders` response includes `X-Printf-Size-System`, reflecting the system that was applied to resolve the order. For mixed-system orders the header reflects the order-level system; per-line overrides are visible only in `resolved_size`.

## Resolution order for bare size labels

When `size_system` is absent from both the line and the order, the API attempts this fallback chain:

1. Order-level `size_system`
2. Account default
3. Fulfilling facility default

Step 3 depends on routing rather than on the payload. If your account can route to more than one facility, the API cannot determine a stable default at request time and returns `400 size_system_ambiguous` (see [Errors](#errors)).

## Errors

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` supplied and the account routes to multiple facilities with different defaults. Add `size_system` to the order or per line. |
| `size_system_implicit` | — | Warning: `size_system` was resolved from the facility default. Becomes an error in **2.6.0**. |

## Saved order templates

Templates are not visible through the API, but they are subject to the same validation rules as live requests. If you have saved templates that omit `size_system` and your account routes to more than one facility, those templates will fail with `size_system_ambiguous` on the next use. Update each template to include `size_system` at the order level before upgrading accounts to 2.4.0.

