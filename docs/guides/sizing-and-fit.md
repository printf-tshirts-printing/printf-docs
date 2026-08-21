---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
owner: devex
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-go
  - printf-java
  - printf-rb
---

# Sizing and fit

Every order line requires a size. As of 2.4.0, you must also tell the API which size ladder you are using — `US`, `EU`, or `JP`. Without it, the API cannot distinguish a JP `XL` (97 cm chest) from a US `XL` (112 cm chest).

## Fields

### On the order

| Field | Type | Required | Description |
|---|---|---|---|
| `size_system` | `"US"` \| `"EU"` \| `"JP"` | Conditionally required (see below) | Default ladder for all lines. Required if your account routes to more than one facility. |

### On each line

| Field | Type | Required | Description |
|---|---|---|---|
| `size` | string | Yes | Label from the chosen ladder (`"S"`, `"M"`, `"L"`, `"XL"`, …) |
| `size_system` | `"US"` \| `"EU"` \| `"JP"` | No | Overrides the order-level `size_system` for this line only |
| `fit` | `"unisex"` \| `"womens"` \| `"mens"` | No | Defaults to `"unisex"` |

### Resolution order

When `size_system` is absent, the API tries each fallback in sequence:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default

Step 4 is only reachable when routing is unambiguous (single facility). Accounts that can route to more than one facility receive `400 size_system_ambiguous` at step 4 instead.

## Example request

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

## Response

Each line in the response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header echoes the resolved system for the order.

## Webhooks

Webhook payloads include the same `resolved_size` object on each line. If you parse size labels from webhook events, add handling for `resolved_size` — it is the canonical post-routing value.

## Ladder reference

| Size | US chest (cm) | EU chest (cm) | JP chest (cm) |
|---|---|---|---|
| S | 97 | 94 | 82 |
| M | 102 | 99 | 87 |
| L | 107 | 104 | 92 |
| XL | 112 | 108 | 97 |

## Errors

| Code | HTTP status | When |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` cannot be resolved and account routes to multiple facilities |
| `size_system_implicit` | warning | `size_system` resolved from facility default; single-facility account only. Becomes an error in 2.6 |

:::warning
If you have saved order templates, they do not carry a `size_system`. Update them before 2.6 ships or submissions will be rejected. See the [migration guide](/guides/sizing-and-fit-migration#migrating-saved-templates).
:::

