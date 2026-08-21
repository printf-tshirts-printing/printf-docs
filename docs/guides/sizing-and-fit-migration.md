---
title: Migration guide — size_system (2.4.0)
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

# Migrating to explicit `size_system`

Orders API 2.4.0 requires you to declare which size ladder you are using. Without it, a JP `XL` (97 cm chest) and a US `XL` (112 cm chest) are indistinguishable — and the wrong one ships.

## Who is affected

| Account type | Behaviour from 2.4.0 | Deadline |
|---|---|---|
| Routes to **more than one facility** | `400 size_system_ambiguous` immediately — orders are rejected today | Fix now |
| Routes to a **single facility** | `size_system_implicit` warning in response; order fulfills using facility default | Fix before 2.6 |
| Already sending `size_system` | No change | — |

If you are on a Keynote plan and are unsure which case applies to you, check `X-Printf-Size-System` in today's responses. A value of `implicit` means you are in the single-facility warning window.

## What to change

Add `size_system` at the order level. If a line uses a different ladder from the order default, override it on that line.

**Before (2.3.x)**

```json
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
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

**After (2.4.0+)**

```diff
 {
   "accountId": "acct_stackfest",
+  "size_system": "US",
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
+      "size_system": "US",
+      "fit": "unisex",
       "quantity": 250,
       "garmentSku": "tee-classic-black"
     }
   ]
 }
```

`size_system` on the line overrides the order-level value for that line only. Omit the line-level field when it matches the order default.

## What you get back

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header echoes the resolved system for the order. Use it to assert that routing resolved as expected.

## Migrating saved order templates

:::danger
Saved templates created before 2.4.0 carry no `size_system`. From 2.6, any template submission that still lacks `size_system` will be rejected with `400 size_system_ambiguous` or `400 size_system_implicit_error` depending on your facility routing.
:::

Templates are not visible through the orders API. To update them:

1. Open **Settings → Order templates** in the dashboard.
2. Edit each template and add `size_system` at the order level.
3. If any line targets a different region's ladder, add `size_system` on that line too.

If you manage templates programmatically, reach out to your account contact — dana.okonkwo@printf.dev (StackFest, KubeSummit, Cloud Native Rodeo) or milos.pavlovic@printf.dev (ObservaCon, ShipItConf) — to export the template list before 2.6 ships.

## Reference: size ladder differences

| Size | US chest (cm) | EU chest (cm) | JP chest (cm) |
|---|---|---|---|
| L | 107 | 104 | 92 |
| XL | 112 | 108 | 97 |

A JP `XL` is 15 cm narrower in the chest than a US `XL`. Resolving to the wrong ladder is a fulfillment error that cannot be corrected after printing.

## Error reference

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; `size_system` is missing or cannot be inferred | Add `size_system` explicitly |
| `size_system_implicit` | — (warning) | Single-facility account; `size_system` resolved from facility default | Add `size_system` before 2.6 |

