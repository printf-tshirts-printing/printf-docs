---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-23
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

As of **2.4.0**, every size label must be paired with a size system. A bare label such as `XL` is ambiguous — a JP `XL` has a 97 cm chest, a US `XL` has 112 cm. The API now enforces this at request time.

## Size systems

| Value | Region | XL chest (cm) |
|---|---|---|
| `US` | United States / Canada | 112 |
| `EU` | Europe | 108 |
| `JP` | Japan | 97 |

## Where to set `size_system`

`size_system` can appear at two levels. The line value takes precedence.

```
Order.size_system          ← default for all lines
  Line.size_system         ← overrides order-level for that line
```

Resolution order when the field is omitted on a line:

1. `lines[].size_system` on the line itself
2. `size_system` on the order
3. Account-level default (set in the dashboard)
4. The fulfilling facility's default — **only for single-facility accounts**

Step 4 produces a `size_system_implicit` warning. Multi-facility accounts are rejected with `400 size_system_ambiguous` at step 4 instead of guessing.

## `fit` per line

Each line now accepts `fit`: `unisex` (default), `womens`, or `mens`. Fit affects both the cut and the size ladder. Always send `fit` explicitly when your design targets a specific cut.

## `resolved_size` in responses

Every response line and every webhook payload now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size` to confirm what was actually applied — especially when relying on order- or account-level defaults.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` contains the resolved system for the request. Useful for logging and debugging without parsing the body.

## Minimal correct request

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

Response excerpt:

```json
{
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "resolved_size": {
        "label": "XL",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 112
      }
    }
  ]
}
```

## Error reference

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account sent a bare size with no `size_system`. Add `size_system` to the line or order. |
| `size_system_implicit` | — (warning) | Single-facility account omitted `size_system`. Will become a hard error in **2.6**. |

## Saved order templates

Templates do not pick up `size_system` automatically. Open each template in the dashboard and add `size_system` (and optionally `fit`) to every line before 2.6 ships. Templates that still have bare sizes at 2.6 will start failing.

