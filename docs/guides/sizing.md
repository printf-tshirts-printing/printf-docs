---
title: Sizing and fit
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/sizes]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

Breaking change in order-api 2.4.0: `size` is no longer interpreted without a size system. If you send a bare label such as `XL` and nothing resolves a system for it, `POST /v2/orders` rejects the order or warns, depending on your account.

This matters because the ladders differ.

| Label | System | Chest |
|---|---|---|
| `XL` | `JP` | 97 cm |
| `XL` | `US` | 112 cm |

## Size systems

`size_system` accepts `US`, `EU` or `JP`. Set it on the order, on a line, or both.

| Field | Where | Values |
|---|---|---|
| `size_system` | Order | `US`, `EU`, `JP` |
| `size_system` | Each line | `US`, `EU`, `JP` |
| `fit` | Each line | For example `unisex` |

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

`accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

## How a bare size is resolved

When a line's `size` is a bare label, Printf looks for a size system in this order and uses the first one it finds.

| Order | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order |
| 3 | The account's size system |
| 4 | The fulfilling facility's default |

Rule 4 depends on routing, not on your payload. For that reason, accounts that can route to more than one facility are not resolved by guess. See the next section.

## Errors and warnings

| Code | Applies to | Result |
|---|---|---|
| `size_system_ambiguous` | Accounts that can route to more than one facility | `400`. The order is not created. |
| `size_system_implicit` | Accounts that route to a single facility | The order is created with a warning. This becomes an error in 2.6. |

Match on the code, not on the message text.

To avoid both, send `size_system` on the order. You then never depend on rule 4.

## What the response tells you

Every response line carries `resolved_size`. The same field appears in webhook payloads. Responses also carry an `X-Printf-Size-System` header.

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Check `resolved_size` on your first order after migrating. It shows the label, system and fit Printf actually used.

## Migrating saved templates

Saved order templates created before 2.4.0 carry no explicit `size_system`. They are invisible from the API, so it is easy to forget they exist. They fail or warn the same way as any other bare-`size` request.

:::warning
A template that still works today may resolve against the facility default, which is not the size system its author had in mind. Review every template, including the ones you rarely use.
:::

For each saved template:

1. Open the template and work out which size system its sizes were written for.
2. Set `size_system` on the template's order. Set it on individual lines only where they differ.
3. Save the template.
4. Place a test order from it and check `resolved_size` on each line.

### Migrating a request

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
      {
        "designId": "dsn_7fa91c",
        "size": "XL",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

### If you do nothing

Multi-facility accounts already get `400 size_system_ambiguous`. Single-facility accounts get a `size_system_implicit` warning now, and an error from 2.6.

