---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders, PATCH /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Sizing and fit

As of order-api 2.4.0, every size label must be accompanied by a size system. A bare `"XL"` on its own is ambiguous — a JP `XL` has a 97 cm chest measurement, a US `XL` has 112 cm. The API enforces this distinction rather than guessing.

## Size systems

| `size_system` value | Covered regions |
|---|---|
| `US` | United States, Canada |
| `EU` | European Union, UK, Australia |
| `JP` | Japan, Korea, South-East Asia |

## Where to specify `size_system`

You can set `size_system` at two levels. The more specific one wins.

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` in the request body | Default for every line that does not specify its own |
| Line | `size_system` inside the `lines[]` entry | Overrides the order-level value for that line |

If neither level is set, the API falls back to your account's configured default, then to the fulfilling facility's default. **This fallback only works when your account routes to a single facility.** Multi-facility accounts that rely on the fallback receive `400 size_system_ambiguous`.

## `fit` per line

Each line also accepts an optional `fit` field:

| `fit` value | Description |
|---|---|
| `unisex` | Standard unisex cut (default) |
| `womens` | Women's ladder |
| `mens` | Men's ladder |
| `youth` | Youth ladder |

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
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "size": "XL",
      "size_system": "US",
      "fit": "unisex"
    }
  ]
}
```

## `resolved_size` in responses

Every line in the response now includes a `resolved_size` object confirming what the API resolved:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header echoes the system that was applied at order level.

## Error and warning reference

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` resolved | Add `size_system` to the order or the line |
| `size_system_implicit` | 200 (warning) | Single-facility account relying on facility default | Add `size_system` before 2.6.0, when this becomes an error |

## Saved order templates

Templates are not visible through the API but they are evaluated on every order they generate. Any template created before 2.4.0 that does not include `size_system` will produce `size_system_implicit` warnings now and will fail outright in 2.6.0. Open each template in the dashboard and add `size_system` to both the order and any lines before upgrading to 2.6.0.

