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

As of **order-api 2.4.0**, every size label must carry an explicit size system. This page covers how `size_system` and `fit` work, how `resolved_size` is returned, and what to do if you are migrating from 2.3 or earlier.

## Why size systems matter

Ladders differ materially across regions. A US XL has a 112 cm chest; a JP XL has a 97 cm chest. Before 2.4.0 the API resolved bare labels by inferring the fulfilling facility's default — a silent rule that produced wrong-size fulfilment whenever routing changed. That behaviour is gone.

## Supported systems

| Value | Region | XL chest (cm) |
|---|---|---|
| `US` | United States | 112 |
| `EU` | Europe | 108 |
| `JP` | Japan | 97 |

## Setting size_system

You can set `size_system` at two levels. The line value takes precedence over the order value.

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

The order-level `size_system` acts as a default for every line that omits its own. Specifying it on both levels (as above) is the safest pattern — it removes any ambiguity if lines are added later.

## Resolution order for bare labels

If a line omits `size_system`, the API resolves in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default (if configured)
4. Fulfilling facility's default — **only for single-facility accounts**

Step 4 is routing-dependent. Accounts that can route to more than one facility are rejected immediately with `400 size_system_ambiguous` rather than resolved by guess. Single-facility accounts resolve at step 4 but receive a `size_system_implicit` warning (see [Deprecation timeline](#deprecation-timeline)).

## fit

`fit` is optional and accepted per line. Valid values: `unisex`, `mens`, `womens`. When omitted the garment's default fit is used.

## resolved_size in responses

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size.chest_cm` to confirm the measurement before displaying it to a buyer. The `X-Printf-Size-System` response header echoes the system applied at the order level.

## Saved order templates

Templates do not appear in the API response, but they are submitted through the same endpoint and are subject to the same rules. Any saved template that omits `size_system` will trigger `size_system_implicit` warnings today and will break in 2.6. Open each template and add `size_system` explicitly.

## Deprecation timeline

| Version | Behaviour for bare labels (single-facility accounts) |
|---|---|
| 2.4.x (now) | Resolves with `size_system_implicit` warning |
| 2.5.x | Warning promoted to prominent header `Deprecation: size_system` |
| 2.6.0 | `400 size_system_required` — hard error, order rejected |

Multi-facility accounts already receive `400 size_system_ambiguous` as of 2.4.0.

