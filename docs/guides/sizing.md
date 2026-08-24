---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of **order-api 2.4.0**, every size label must carry an explicit size system. Ladders differ materially across systems — a JP `XL` has a 97 cm chest; a US `XL` has 112 cm — so the API no longer resolves an ambiguous size by guessing.

## Size systems

| Value | Region | Notes |
|---|---|---|
| `US` | United States | Default for most North American accounts |
| `EU` | Europe | EN 13402 grading |
| `JP` | Japan | JIS grading; runs smaller than US/EU |

## Where to set `size_system`

`size_system` can appear at two levels. The line-level value wins; the order-level value is the fallback for any line that omits it.

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

The `size_system` on the line (`US`) overrides the order-level `size_system`. If you set `size_system` at the order level and omit it on a line, the order-level value applies to that line.

## Fit

`fit` is a per-line field introduced in 2.4.0. It is optional but recommended when your catalog includes gendered cuts.

Common values: `unisex`, `women`, `men`. The accepted set is garment-specific — consult your garment catalog or the `GET /v2/garments/{garmentSku}` endpoint.

## `resolved_size` in responses

Every line item in order responses and webhook payloads now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` to verify that the system and size label combination matches your expectation before confirming production.

## `X-Printf-Size-System` response header

The `X-Printf-Size-System` response header reports the effective size system applied to the order. When multiple systems are present across lines, the header value is `mixed`.

## Resolution order

The API resolves `size_system` using the following precedence, from highest to lowest:

1. `size_system` on the individual line
2. `size_system` on the order body
3. `size_system` set on the account record
4. The fulfilling facility's default

Rule 4 depends on routing, not on the payload. Accounts that can route to **more than one facility** with different defaults are rejected at rule 4 with `400 size_system_ambiguous`. You must supply `size_system` explicitly at the order or line level to resolve the ambiguity.

Single-facility accounts that reach rule 4 succeed but receive a `size_system_implicit` warning in the response body. **This warning becomes an error (`400`) in 2.6.** Set `size_system` explicitly before then.

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities with different defaults; `size_system` required |
| `size_system_implicit` | — (warning) | `size_system` resolved from facility default; will become `400` in 2.6 |
| `size_not_found` | 422 | `size` label does not exist in the resolved `size_system` |

