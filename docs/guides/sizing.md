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

As of **order-api 2.4**, every size label must be paired with an explicit size system, or must resolve to one through the fallback chain described below. Bare labels without a resolvable system are rejected.

## Size systems

| `size_system` value | Region | XL chest (cm) |
|---|---|---|
| `US` | United States / Canada | 112 |
| `EU` | Europe | 107 |
| `JP` | Japan | 97 |

Ladders differ materially. A JP `XL` and a US `XL` are not the same garment. Always set the system explicitly; do not rely on implicit resolution in new integrations.

## Setting the size system

You can set `size_system` at two levels:

- **Order level** — applies to every line that does not override it.
- **Line level** — overrides the order-level value for that line only.

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

Each line item accepts a `fit` value:

| Value | Description |
|---|---|
| `unisex` | Standard unisex cut |
| `fitted` | Contoured cut |
| `relaxed` | Relaxed / oversized cut |

`fit` is optional. When omitted, the garment's default fit is used.

## The `resolved_size` response object

Every line in the order response and in webhook payloads includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size.chest_cm` to verify that the system resolved as you intended, especially when mixing US and EU lines in a single order.

## The `X-Printf-Size-System` response header

The response includes a header echoing the effective size system for the order:

```
X-Printf-Size-System: US
```

When lines use mixed systems the header reflects the order-level value. Log this header in your integration for auditability.

## Fallback resolution chain

When `size_system` is absent on a line, resolution proceeds in this order:

1. `size_system` on the line
2. `size_system` on the order
3. The size system configured on the account
4. The fulfilling facility's default

Step 4 depends on routing, not on the payload. That makes it ambiguous for accounts that can route to more than one facility.

## Error and warning codes

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; no system could be resolved without guessing. | Set `size_system` explicitly on the order or per line. |
| `size_system_implicit` | — (warning) | System resolved via account or facility default. Becomes a hard error in **2.6**. | Set `size_system` explicitly before upgrading to 2.6. |

## Saved order templates

Templates do not automatically inherit `size_system`. If you have saved templates that omit the field, they will trigger `size_system_implicit` warnings today and will fail in 2.6. Edit each template to add `size_system` at the order level or per line.
