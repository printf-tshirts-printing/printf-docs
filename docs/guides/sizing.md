---
title: Sizing and fit
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/sizes]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

Breaking, as of Orders API 2.4.0: `size` is no longer interpreted without a size system. If your account can route to more than one facility and you send a bare size label such as `XL`, `POST /v2/orders` returns `400 size_system_ambiguous`. If your account routes to a single facility, the order succeeds with a `size_system_implicit` warning, which becomes an error in 2.6.

## Why a label is not enough

The same label means a different garment on each ladder.

| Label | System | Chest |
|---|---|---|
| `XL` | `US` | 112 cm |
| `XL` | `JP` | 97 cm |

## Set the size system

| Field | Where | Values | Notes |
|---|---|---|---|
| `size_system` | Order | `US`, `EU`, `JP` | Applies to every line that does not set its own. |
| `size_system` | Each line | `US`, `EU`, `JP` | Overrides the order value for that line. |
| `fit` | Each line | for example `unisex` | Returned in `resolved_size`. |

`POST /v2/orders`

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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

`accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

## How a bare label resolves

If a line has no `size_system`, Printf checks these in order and uses the first that is set.

| Order | Source |
|---|---|
| 1 | The line's `size_system` |
| 2 | The order's `size_system` |
| 3 | The account |
| 4 | The fulfilling facility's default |

Rule 4 depends on how the order is routed, not on your payload. For that reason a bare label that reaches rule 4 is handled by account type:

| Code | Applies to | Result |
|---|---|---|
| `size_system_ambiguous` | Accounts that can route to more than one facility | `400`. The order is rejected. |
| `size_system_implicit` | Accounts that route to a single facility | Warning. The order succeeds. Becomes an error in 2.6. |

Send `size_system` on the order to avoid both.

## Read the result

Each response line carries `resolved_size`, so you can confirm what was ordered.

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Responses also carry an `X-Printf-Size-System` header. Webhook payloads carry `resolved_size` as well.

## Migrating saved templates

Saved order templates created before 2.4.0 carry no explicit `size_system`. They resolve through rules 3 and 4 above, so a multi-facility account gets `400 size_system_ambiguous` and a single-facility account gets `size_system_implicit`.

Templates do not appear in API responses, so list them from wherever you manage them and from any automation that submits them. For each template:

1. Add `size_system` to the template. Use the order-level field when every line uses one ladder, or set it per line when the template mixes ladders.
2. Submit one order from the template and check `resolved_size` on each response line.
3. Repeat for templates you rarely use. They fail the same way.

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
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
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

:::warning
A single-facility account will not see an error for a template without `size_system` until 2.6. Until then the order succeeds using the facility default, which may be a different ladder from the one you intended.
:::

