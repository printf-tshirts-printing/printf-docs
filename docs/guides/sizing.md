---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Sizing and fit

As of order-api **2.4.0**, every size label must be accompanied by a size system. Bare labels (e.g. `"XL"` with no `size_system`) are no longer silently resolved and will produce a `400 size_system_ambiguous` error for multi-facility accounts.

## Size systems

| `size_system` value | Region |
|---|---|
| `US` | United States |
| `EU` | Europe |
| `JP` | Japan |

Systems differ materially. A JP `XL` chest is **97 cm**; a US `XL` chest is **112 cm**. Always specify the system explicitly — do not rely on account or facility defaults.

## Where to set `size_system`

`size_system` can be set at two levels and resolves from most-specific to least-specific:

1. **Per line** — set `size_system` on the individual `lines[]` item. Takes precedence over everything.
2. **Per order** — set `size_system` at the top level of the request. Applies to all lines that do not carry their own value.

If neither is present, the API attempts to fall back to the account default, then the fulfilling facility's default. Accounts that can route to more than one facility cannot be resolved this way and receive `400 size_system_ambiguous`.

## `fit`

`fit` is set per line. Accepted values are `unisex`, `mens`, and `womens`. When omitted, the garment SKU's default fit is used.

## Example — explicit size system per line

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
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

## `resolved_size` in the response

Every line in a successful response includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The same object appears in webhook payloads for `order.created` and `order.updated`.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reflects the size system that was applied to resolve sizes for the request. Use it to confirm the system your payload implied.

## Warnings and errors

| Code | Type | Meaning |
|---|---|---|
| `size_system_ambiguous` | `400` error | No `size_system` on the request and the account can route to more than one facility. Request rejected. |
| `size_system_implicit` | Warning (2.4–2.5) | No `size_system` on the request; resolved via single-facility account default. Becomes a `400` error in 2.6.0. |

To resolve `size_system_ambiguous`: add `size_system` to the order or to each line. To suppress `size_system_implicit`: do the same before 2.6.0.

