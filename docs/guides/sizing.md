---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-21
owner: platform-docs
covers_endpoints:
  - POST /v2/orders
  - POST /v2/orders/bulk
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of **API 2.4.0**, size labels are explicit and unambiguous. Every order and line item carries a declared size system; the API tells you exactly what it resolved.

## Size systems

| Code | Ladder | US XL chest (cm) | JP XL chest (cm) |
|---|---|---|---|
| `US` | US / North America | 112 | — |
| `EU` | European | 107 | — |
| `JP` | Japanese | — | 97 |

A JP `XL` and a US `XL` differ by 15 cm. Omitting `size_system` and relying on implicit resolution **will become an error in 2.6** — set it explicitly now.

## Declaring `size_system`

`size_system` can be set at two levels. The more specific value wins.

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` (top-level) | Default for all lines in this order |
| Line | `lines[].size_system` | Overrides the order-level value for this line only |

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

## The `fit` field

`fit` is set per line item. Accepted values depend on the garment SKU, but common values are `unisex`, `womens`, and `mens`.

## What the API gives back

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response header `X-Printf-Size-System` reflects the system applied to resolve any bare labels on the request.

## Resolution order for bare size labels

If you omit `size_system` on a line, the API resolves it in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default

**Step 4 is routing-dependent.** If your account can route to more than one facility, the API cannot determine which facility will fulfil the order at request time. The request is rejected with `400 size_system_ambiguous`. Set `size_system` explicitly to avoid this.

Single-facility accounts reach step 4 successfully but receive a `size_system_implicit` warning in the response. That warning becomes `400 size_system_implicit` in **2.6**.

## Saved order templates

Templates do not automatically inherit `size_system`. Any template that omits `size_system` will trigger `size_system_implicit` warnings today and will fail in 2.6. Open each template in the dashboard or via the API and add `size_system` explicitly.

