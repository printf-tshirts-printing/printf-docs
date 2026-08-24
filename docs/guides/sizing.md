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
  - printf-rb
  - printf-go
---

# Sizing and fit

As of **order-api 2.4.0**, every size label must be accompanied by an explicit size system. The API no longer resolves bare labels by guessing based on routing. This page covers `size_system`, `fit`, and the `resolved_size` field you receive on every response line.

## Size systems

| `size_system` value | Region | XL chest (cm) |
|---|---|---|
| `US` | United States / Canada | 112 |
| `EU` | Europe | 104 |
| `JP` | Japan | 97 |

A JP `XL` and a US `XL` differ by 15 cm. The API will not pick one for you.

## Setting the size system

You can set `size_system` at two levels. The per-line value always wins.

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
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "EU",
      "fit": "womens",
      "quantity": 100
    }
  ]
}
```

The two `XL` lines above resolve to different physical garments. Without `size_system` on each line you would have to rely on the order-level default, and if routing can reach more than one facility the request is rejected with `400 size_system_ambiguous`.

## Resolution order

When a line does not carry its own `size_system`, the API walks this chain and uses the first value it finds:

1. `size_system` on the line
2. `size_system` on the order root
3. `size_system` on the account profile
4. Default system of the fulfilling facility

Step 4 is only reached for single-facility accounts. If it is reached, the response includes a `size_system_implicit` warning. That warning becomes a hard `400` error in **2.6.0**.

## The `fit` field

`fit` is set per line. Accepted values:

| Value | Description |
|---|---|
| `unisex` | Standard relaxed cut (default when omitted) |
| `mens` | Narrower shoulder, longer body |
| `womens` | Tapered waist, shorter body |

`fit` is returned verbatim in `resolved_size` so you can confirm what was cut.

## The `resolved_size` response object

Every line in a successful response and every webhook payload now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

| Field | Type | Description |
|---|---|---|
| `label` | string | The canonical label after normalisation |
| `system` | string | The system actually used — useful when you relied on order- or account-level defaults |
| `fit` | string | The fit used |
| `chest_cm` | number | Physical chest measurement for this label in this system |

Use `chest_cm` in your QA checks to catch system mismatches before garments ship.

## The `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reports the system used for the majority of lines in the order. When lines carry mixed systems this header reflects the order-level default or `mixed` if no single system dominates.

```
X-Printf-Size-System: US
```

## Error reference

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Bare size reached routing; more than one facility could fulfil and each uses a different ladder | Add `size_system` explicitly to every line |
| `size_system_implicit` | — (warning) | Size resolved by facility default; works today, error in 2.6.0 | Add `size_system` explicitly; do not rely on facility default |

## Saved order templates

Templates are not surfaced in the API response, but they are validated on submission. If a template was created before 2.4.0 and carries bare size labels, the first submission after upgrade will return `size_system_ambiguous` or `size_system_implicit` depending on your account routing configuration. Open each template and add `size_system` to the order root and to every line before 2.6.0.

