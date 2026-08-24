---
title: Sizing and fit
section: Guides
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

> **2.4.0 breaking change.** `size` is no longer interpreted without a declared size system. Read this page before sending orders.

As of order-api 2.4.0, every size label must be anchored to a size system. A bare `"size": "XL"` with no system context is ambiguous: a JP XL has a 97 cm chest; a US XL has a 112 cm chest. The API now rejects or warns instead of guessing.

## Size systems

| Value | Region | Notes |
|---|---|---|
| `US` | United States | Default for most North American facilities |
| `EU` | Europe | |
| `JP` | Japan | Ladders shift materially vs US/EU |

Declare `size_system` at the order level, the line level, or both. When both are present, the line-level value wins.

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

Mixing systems across lines in the same order is supported — set `size_system` per line.

## Fit

The optional `fit` field on each line accepts values such as `unisex`, `mens`, and `womens`. When omitted, the facility's default fit for the garment is applied.

## Fallback resolution chain

When `size_system` is omitted on a line, the API walks this chain and uses the first value it finds:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default size system (set in account settings)
4. Fulfilling facility's default size system

Step 4 depends on order routing, not on the payload. **If your account can route to more than one facility, step 4 is unavailable** — the API returns `400 size_system_ambiguous` rather than guess. Set `size_system` explicitly to avoid this.

Single-facility accounts that reach step 4 receive a `size_system_implicit` warning in the response. This warning becomes a hard error in 2.6.

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

Use `chest_cm` to verify the physical measurement that will be fulfilled. This field is also present on webhook payloads for order confirmation events.

## Response header

`X-Printf-Size-System` is returned on every order response and echoes the size system applied at the order level. Useful for logging and debugging without parsing the body.

## Size ladder reference

| Label | US chest (cm) | EU chest (cm) | JP chest (cm) |
|---|---|---|---|
| S | 88 | 88 | 84 |
| M | 96 | 96 | 90 |
| L | 104 | 104 | 97 |
| XL | 112 | 112 | 97 |
| 2XL | 120 | 120 | 104 |

> JP ladders compress at the top end. Verify `chest_cm` in `resolved_size` rather than mapping labels 1:1 across systems.

## Common errors

| Code | Status | Cause | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no size system resolved | Set `size_system` on the order or line |
| `size_system_implicit` | warning | Single-facility account, size system resolved from facility default | Set `size_system` explicitly before 2.6 |

