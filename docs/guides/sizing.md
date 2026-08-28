---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-28
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

As of Orders API 2.4.0, every order must carry an explicit size system. A bare size label — `XL`, `L`, `M` — means nothing without one. This page explains how to supply it, what the response carries back, and what to do with any existing templates or integrations.

:::danger
Accounts that route to more than one facility will receive `400 size_system_ambiguous` on any request that omits `size_system`. This is a breaking change from 2.3 behaviour, where the fulfilling facility's default was used silently.
:::

## Size systems

| System | Value | US XL chest | JP XL chest | EU XL chest |
|--------|-------|-------------|-------------|-------------|
| United States | `US` | 112 cm | — | — |
| Japan | `JP` | — | 97 cm | — |
| Europe | `EU` | — | — | varies by brand |

Ladders differ materially. A JP `XL` is 97 cm; a US `XL` is 112 cm. Supply the wrong system and the wrong garment ships.

## Supplying `size_system`

`size_system` can be set at the order level, at the line level, or both. Line-level takes precedence over order-level.

### Order-level (applies to every line)

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
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### Per-line (overrides order-level)

Use line-level `size_system` when a single order mixes garments intended for different markets.

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
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
    },
    {
      "designId": "dsn_9bc22d",
      "size": "XL",
      "size_system": "JP",
      "fit": "fitted",
      "quantity": 100,
      "garmentSku": "tee-slim-white"
    }
  ]
}
```

## `fit`

Optional per-line field. Accepted values: `unisex`, `fitted`, `relaxed`. Defaults to the garment SKU's catalogue default when omitted.

## Response fields

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

The response also includes the `X-Printf-Size-System` header echoing the system used to resolve the order.

Webhook payloads carry the same `resolved_size` structure on each line.

## Resolution precedence

When `size_system` is omitted:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default ← **this step is now unavailable for multi-facility accounts**

Accounts routable to more than one facility cannot safely resolve at step 4 because the facility is chosen after the payload is submitted. These accounts receive `400 size_system_ambiguous`.

Single-facility accounts still resolve at step 4 but receive a `size_system_implicit` warning. This warning becomes an error in **2.6**.

## Error reference

| Code | HTTP status | Meaning |
|------|-------------|----------|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; `size_system` is required |
| `size_system_implicit` | — (warning) | Single-facility account resolved via facility default; add `size_system` before 2.6 |

## Migrating saved templates {#migrating-saved-templates}

Saved order templates created before 2.4.0 carry no `size_system`. They will continue to resolve against your account or facility default for now — but if your account routes to more than one facility, they will fail immediately with `size_system_ambiguous`. All templates will fail in **2.6** when `size_system_implicit` becomes an error.

**Who is affected:** any account using saved order templates, and any account that routes to more than one facility.

**What to do:** fetch each template, add `size_system` at the order level or per line, and save it back.

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
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

**What happens if you do nothing:** multi-facility accounts break immediately on 2.4.0. Single-facility accounts receive `size_system_implicit` warnings until **2.6**, at which point requests without an explicit `size_system` are rejected.

