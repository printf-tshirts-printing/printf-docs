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

**As of order-api 2.4.0, every size must be paired with an explicit `size_system`.** Bare size labels without a system are rejected for multi-facility accounts and warn for single-facility accounts. See [Migration — size_system required](#migration) below.

## Size systems

Printf fulfillment supports three size systems. Ladders differ materially — do not assume equivalence.

| System | `size_system` value | US XL chest (cm) | JP XL chest (cm) |
|---|---|---|---|
| United States | `US` | 112 | — |
| European | `EU` | 107 | — |
| Japanese | `JP` | — | 97 |

Always specify the system that matches your customer base. A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. Substituting one for the other silently produces the wrong garment.

## Setting `size_system`

`size_system` can be set at two levels. The line-level value takes precedence over the order-level value.

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` (top-level) | Applies to all lines that omit their own `size_system` |
| Line | `size_system` (inside `lines[]`) | Overrides the order-level value for that line |

Setting it at the order level is the simplest approach when all lines use the same system:

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
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

To mix systems within one order, set `size_system` per line:

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## The `fit` field

`fit` is set per line. It controls the cut of the garment. Available values depend on the garment SKU — consult the product catalog for valid options. `unisex` is the most common value.

## `resolved_size` in responses

Every line in the response and in webhook payloads now includes a `resolved_size` object confirming the system and physical dimensions applied:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Compare `resolved_size.chest_cm` against your design brief before accepting a proof. Discrepancies here indicate a system mismatch.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reports the size system ultimately applied to the order. Useful for logging and for asserting the correct system in integration tests.

## Resolution order for bare size labels

If a line omits `size_system`, resolution falls back in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account-level default
4. Fulfilling facility's default

Step 4 depends on routing, not on the payload. **Accounts that can route to more than one facility are rejected with `400 size_system_ambiguous` at step 4.** Single-facility accounts receive a `size_system_implicit` warning today; this becomes a hard error in **2.6**.

Do not rely on fallback past step 2. Set `size_system` explicitly.

---

## Migration — size_system required {#migration}

**What changed:** `size` is no longer interpreted without a system. Omitting `size_system` on a multi-facility account returns `400 size_system_ambiguous`. Omitting it on a single-facility account returns a `size_system_implicit` warning (hard error in 2.6).

**Who is affected:** Any integration that posts a `size` field without a matching `size_system` at the order or line level.

**What to do:**

Before (2.3 and earlier — no longer accepted for multi-facility accounts):

```json
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
      "size": "XL",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

After (2.4+ — required for all accounts, eliminates warning):

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

**If you do nothing:** Multi-facility accounts receive `400 size_system_ambiguous` immediately and orders fail. Single-facility accounts receive `size_system_implicit` warnings now; those become `400` errors in **2.6**.

**Saved order templates:** Templates that do not include `size_system` are affected. The API applies the same fallback logic to templates at submission time. Audit your saved templates and add `size_system` to each line.
