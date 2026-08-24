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

As of **order-api 2.4.0**, every size label must be paired with a size system. A bare label like `XL` means different physical measurements in different systems, and the API no longer guesses.

## Size systems

| `size_system` value | Region | Example: `XL` chest |
|---|---|---|
| `US` | United States | 112 cm |
| `EU` | Europe | 106 cm |
| `JP` | Japan | 97 cm |

Set `size_system` at the order level as a default, then override it per line when a single order mixes systems.

## Fit options

| `fit` value | Description |
|---|---|
| `unisex` | Straight cut, default |
| `mens` | Narrower waist, longer body |
| `womens` | Tapered cut |

`fit` is set per line. When omitted the facility's default fit is used.

## Request shape

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

The `size_system` on the line takes precedence over the order-level value. Provide both for clarity and to protect against future routing changes.

## Response: `resolved_size`

Every response line and webhook payload now includes a `resolved_size` object so you can verify what the fulfillment system actually booked:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response header `X-Printf-Size-System` reports the single system used to resolve all lines in the request. If lines have mixed systems the header is omitted.

## Resolution fallback chain

When `size_system` is absent on a line, the API tries each level in order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default

Step 4 depends on routing. Accounts that can route to **more than one facility** are rejected immediately with `400 size_system_ambiguous` — there is no safe guess. Accounts routed to a single facility receive a `size_system_implicit` warning today; that warning becomes a hard error in **2.6**.

## Error codes

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` missing and account can reach multiple facilities. Hard error now. |
| `size_system_implicit` | — (warning) | `size_system` missing but account has exactly one reachable facility. Becomes hard 400 in 2.6. |

## Saved order templates

Templates are not validated when they are saved — they are validated at dispatch time. Any template created before 2.4.0 that omits `size_system` will begin failing or warning immediately. Review and update all templates before your next dispatch run.

