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

As of **2.4.0**, every size label must be anchored to a size system. Bare size labels without a system are deprecated and become an error in 2.6.

## Size systems

| `size_system` | Region | US `XL` chest equivalent |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 112 cm (different ladder rungs) |
| `JP` | Japan | 97 cm |

Ladders differ materially between systems. A JP `XL` is not the same garment as a US `XL`. Always supply the system your customers are buying in.

## Where to set `size_system`

You can set `size_system` at two levels:

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` (top-level) | Default for every line in this order |
| Line | `size_system` (inside each line object) | Overrides the order-level value for that line only |

Line-level always wins. If you mix systems in a single order, set the order-level to the majority system and override individual lines.

## `fit`

`fit` is a per-line field. Accepted values depend on the garment SKU but typically include `unisex`, `fitted`, and `relaxed`. If omitted, the garment's catalogue default applies.

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

Use `resolved_size` to verify that the system and fit the API resolved match what your customer ordered. The `chest_cm` value is the authoritative dimension used for fulfillment.

## `X-Printf-Size-System` response header

The response includes an `X-Printf-Size-System` header containing the effective size system applied to the order. Useful for logging and debugging without parsing the body.

## Resolution fallback and errors

When `size_system` is absent on a line, the API resolves it in order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default

Step 4 is routing-dependent. If your account can route to more than one facility, the API cannot safely apply step 4 and returns `400 size_system_ambiguous`. Set `size_system` explicitly to fix this.

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in the response. **This warning becomes `400` in 2.6.** Fix it before upgrading.

## Saved order templates

Templates are not modified automatically. If you have saved templates that omit `size_system`, they will produce `size_system_implicit` warnings now and will fail in 2.6. Open each template and add `size_system` at the order level or on each line.
