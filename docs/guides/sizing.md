---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
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

As of **2.4.0**, every size label is resolved against an explicit size system. A bare `"XL"` means different things in different markets — a JP `XL` chest is 97 cm, a US `XL` chest is 112 cm. The API now requires you to say which you mean.

## Size systems

| Value | Region | XL chest (cm) |
|---|---|---|
| `US` | United States / Canada | 112 |
| `EU` | Europe | 104 |
| `JP` | Japan | 97 |

## Setting `size_system`

You can set `size_system` at two levels. The line-level value wins if both are present.

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

Resolution order when `size_system` is absent at a given level:

1. Line `size_system`
2. Order `size_system`
3. Account default (set in dashboard or via account API)
4. Fulfilling facility's default

Step 4 depends on routing, not on your payload. If your account can route to more than one facility, the API cannot determine which facility's default to apply and rejects the request with `400 size_system_ambiguous`. See [Migration: adding `size_system`](#migration-adding-size_system) below.

## `fit`

`fit` is a per-line field. Accepted values depend on the `garmentSku`; common values:

| Value | Meaning |
|---|---|
| `unisex` | Unisex cut |
| `mens` | Men's cut |
| `womens` | Women's cut |

Omitting `fit` leaves it to the garment's own default. Include it whenever you need a deterministic result.

## `resolved_size` in responses

Every response line and webhook payload now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size.chest_cm` to verify that the size that will be printed matches your expectation before the order reaches production.

## `X-Printf-Size-System` response header

Every response to `POST /v2/orders` includes:

```
X-Printf-Size-System: US
```

The value is the effective size system that was applied at the order level. Use it for logging and debugging without parsing the body.

## Migration: adding `size_system`

See the [Migration note](../migration/2.4.0-size-system.md) for a full before/after diff and the rollout timeline.

