---
title: Sizing and fit
section: guides
last_reviewed: 2026-09-24
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/orders/{orderId}]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

As of Orders API 2.4.0, every order must declare a size system. A bare size label such as `XL` means different things in different markets — a JP `XL` chest is 97 cm; a US `XL` is 112 cm. Resolving `XL` against the wrong ladder ships the wrong garment.

:::danger
Accounts that can route to more than one facility will receive `400 size_system_ambiguous` if `size_system` is omitted. There is no fallback. Add `size_system` to every order before deploying against 2.4.0 or later.
:::

## Supported size systems

| Value | Market | XL chest |
|-------|--------|-----------|
| `US` | United States / Canada | 112 cm |
| `EU` | European standard | 107 cm |
| `JP` | Japan | 97 cm |

## Request shape

`size_system` is set at the order level and may be overridden per line. `fit` is per-line only.

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

### Resolution precedence

When `size_system` is set at multiple levels, the most specific value wins:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default *(deprecated — triggers `size_system_implicit` warning; error in 2.6)*
4. Facility default *(removed for multi-facility accounts; see `size_system_ambiguous` below)*

## Response

Each line in the response now carries `resolved_size`:

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

The `X-Printf-Size-System` response header reflects the system applied to the order.

Webhook payloads include the same `resolved_size` object on each line item.

## Error codes

| Code | HTTP status | Meaning |
|------|-------------|---------|
| `size_system_ambiguous` | 400 | No `size_system` supplied and account routes to more than one facility. |
| `size_system_implicit` | — (warning) | No `size_system` supplied; resolved from account or facility default. Becomes an error in 2.6. |

## Migrating saved templates {#migrating-saved-templates}

This is the most common source of `size_system_ambiguous` errors after upgrading.

**What changed.** `size_system` is now required when an account routes to more than one facility. The API no longer falls back to the fulfilling facility's default.

**Who is affected.** Accounts configured to route to more than one facility that have saved order templates created before 2.4.0. Those templates carry no explicit `size_system` and will be rejected on submission.

**What to do.** Retrieve each saved template and add `size_system` at the order level. If individual lines target different markets, set `size_system` per line as well.

Before:

```diff
  {
    "accountId": "acct_stackfest",
    "lines": [
      { "designId": "dsn_7fa91c", "size": "XL", "quantity": 250, "garmentSku": "tee-classic-black" }
    ]
  }
```

After:

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
    "lines": [
      { "designId": "dsn_7fa91c", "size": "XL",
+       "size_system": "US",
+       "fit": "unisex",
        "quantity": 250, "garmentSku": "tee-classic-black" }
    ]
  }
```

**What happens if you do nothing.** Multi-facility accounts will receive `400 size_system_ambiguous` on every order submission that omits `size_system`. Single-facility accounts will receive a `size_system_implicit` warning today and an error starting in 2.6.
