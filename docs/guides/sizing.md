---
title: Sizing and fit
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

As of Orders API 2.4.0, every order must carry an explicit `size_system`. Bare size labels are no longer resolved by the fulfilling facility's default.

:::danger
Accounts that can route to more than one facility will receive `400 size_system_ambiguous` if `size_system` is absent. Single-facility accounts receive a `size_system_implicit` warning today; this becomes a hard error in 2.6.
:::

## Supported size systems

| Value | Standard | Example XL chest |
|---|---|---|
| `US` | US/CA unisex | 112 cm |
| `EU` | European | 104 cm |
| `JP` | Japanese Industrial Standard | 97 cm |

A JP `XL` and a US `XL` differ by 15 cm. Omitting `size_system` does not mean "any system" — it means "guess", and guessing is now an error for multi-facility accounts.

## Specifying size system

`size_system` may be set at the order level, at the line level, or both. The line value takes precedence over the order value.

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

The line-level `size_system` overrides the order-level value for that line. Use the order-level field when every line shares a system; use line-level when mixing systems in a single order.

## fit

`fit` is accepted per line. Supported values depend on the garment SKU. Omitting `fit` leaves the fit at the SKU's default — check the product catalogue if you are unsure what that default is.

## resolved_size

Every line in the response carries `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

`resolved_size` also appears in webhook payloads for `order.created` and `order.updated` events.

## X-Printf-Size-System response header

The response carries `X-Printf-Size-System` reflecting the system applied to the order. When lines carry mixed systems, the header reflects the order-level system; per-line resolution is in `resolved_size`.

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` on the order or line; account routes to more than one facility. Add `size_system`. |
| `size_system_implicit` | Warning (200) | No `size_system`; single-facility account. Will become 400 in 2.6. |

---

## Migrating to explicit size_system {#migrating-to-explicit-size-system}

**What changed.** `POST /v2/orders` no longer resolves a bare size label using the fulfilling facility's default. You must supply `size_system` on the order or on each line.

**Who is affected.** Any integration that submits orders without an explicit `size_system`, and any saved order template created before 2.4.0. Multi-facility accounts receive a hard `400` today. Single-facility accounts receive a warning now and a hard error in 2.6.

**What to do.**

```diff
  POST /v2/orders
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
+       "size_system": "US",
+       "fit": "unisex",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

**Saved order templates.** Templates created before 2.4.0 carry no `size_system`. They will resolve via the implicit-warning path today and fail in 2.6. Retrieve each template, add `size_system` at the order level (and optionally per line), and save it back. There is no bulk migration endpoint — each template must be updated individually.

**What happens if you do nothing.** Multi-facility accounts fail immediately with `400 size_system_ambiguous`. Single-facility accounts continue to work with a `size_system_implicit` warning in the response until 2.6, when the same request becomes a `400`.
