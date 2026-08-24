---
title: Bulk orders and templates
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

# Bulk orders and templates

This guide covers sending large order batches and maintaining saved order templates. If you use templates, read the [size system migration note](#size-system-migration-for-templates) before your next dispatch run — templates created before 2.4.0 require a one-time update.

## Structuring a bulk request

Each order in a bulk run is a separate `POST /v2/orders` call. Required fields on every call:

| Field | Type | Notes |
|---|---|---|
| `accountId` | string | Your account identifier |
| `destination` | object | Full address — see below |
| `lines[].designId` | string | Design to print |
| `lines[].garmentSku` | string | Garment SKU — never abbreviate to `sku` |
| `lines[].quantity` | integer | Units per line |

As of **2.4.0**, `size_system` is also strongly recommended at both order and line level (see below).

### Minimal valid 2.4.0 order

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

## Processing bulk responses

Every successful response line now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Log or store `resolved_size` alongside each order. It is the authoritative record of what the fulfillment system booked and the first thing support will ask for if a garment arrives in the wrong dimensions.

The `X-Printf-Size-System` response header carries the system used for the entire order. Validate it in your bulk pipeline to catch integration mismatches early.

## Size system migration for templates

> ⚠️ **Action required before next dispatch run.**

Templates are validated at dispatch time, not when they are saved. Any template that omits `size_system` will now:

- **Fail immediately** with `400 size_system_ambiguous` if your account can route to more than one facility.
- **Warn** with `size_system_implicit` if your account routes to a single facility. This warning becomes a hard error in **2.6**.

To update a template, re-save it with `size_system` added at both the order level and each line level. The API does not patch templates in place — overwrite the whole template object.

See the [Sizing and fit guide](docs/guides/sizing.md) for the full field reference and resolution fallback chain.

## Error handling in bulk runs

For bulk pipelines, treat these codes as terminal for that order — do not retry without fixing the payload:

| Code | HTTP status | Fix |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to the order and each line |
| `size_system_implicit` | warning (400 in 2.6) | Add `size_system` proactively |

All other 4xx codes follow the same terminal rule. 5xx codes are safe to retry with exponential backoff.

