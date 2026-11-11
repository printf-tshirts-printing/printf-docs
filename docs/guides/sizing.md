---
title: Sizing and fit
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

Breaking change in order-api 2.4.0: `size` is no longer interpreted without a system. If your account can route to more than one facility, an order with a bare size label is rejected with `400 size_system_ambiguous`. If you have saved order templates, see [Migrating saved templates](#migrating-saved-templates).

## Why the system matters

Size ladders differ between systems. The same label is a different garment.

| Label | System | Chest |
|---|---|---|
| `XL` | `US` | 112 cm |
| `XL` | `JP` | 97 cm |

## Fields

| Field | Where | Values | Notes |
|---|---|---|---|
| `size_system` | Order and each line | `US`, `EU`, `JP` | A line value overrides the order value. |
| `fit` | Each line | for example `unisex` | Resolved and echoed back in `resolved_size`. |
| `resolved_size` | Each response line and webhook payload | object | What was actually ordered. |
| `X-Printf-Size-System` | Response header | `US`, `EU`, `JP` | The system applied to the order. |

## How a bare size resolves

If a line has only `size`, the system is taken from the first source below that has a value.

| Order | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order |
| 3 | The account's `size_system` |
| 4 | The fulfilling facility's default |

The last rule depends on routing, not on your payload. That is why the API does not apply it for accounts that can route to more than one facility.

## Error and warning codes

| Code | Who gets it | Meaning |
|---|---|---|
| `size_system_ambiguous` | Accounts that can route to more than one facility (`400`) | A bare size could not be resolved without guessing. Send `size_system`. |
| `size_system_implicit` | Accounts that route to a single facility (warning) | The size resolved to the facility default. Becomes an error in 2.6. |

## Example

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

Each response line carries `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Compare `chest_cm` with what you meant to order before the order is fulfilled.

## Migrating saved templates

Saved order templates created before 2.4.0 carry no explicit `size_system`. They are invisible from the API, so they are easy to forget.

:::warning
A saved template with no `size_system` fails with `400 size_system_ambiguous` on a multi-facility account. On a single-facility account it resolves to the facility default and returns `size_system_implicit`, and that becomes an error in 2.6. If the facility default is not the system you meant, the order arrives in the wrong size.
:::

For each saved template:

1. Set `size_system` on the template's order.
2. Set `size_system` on any line that uses a different system from the order.
3. Place a test order from the template and check `resolved_size` on every line.

Before and after:

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
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
      { "designId": "dsn_7fa91c", "size": "XL", "quantity": 250, "garmentSku": "tee-classic-black" }
    ]
  }
```

