---
title: Size system migration — 2.4.0
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks: []
---

# Size system migration — 2.4.0

## What changed

API version 2.4.0 adds explicit size system support to orders. The fields `size_system` (order and line level) and `fit` (line level) are new. Response lines and webhook payloads now include a `resolved_size` object. A new response header, `X-Printf-Size-System`, reports the system that was applied.

Previously, bare size labels such as `"L"` or `"XL"` were resolved by the fulfilling facility's internal default. That default varied silently by facility. **Two identical payloads routed to different facilities could produce garments with different chest measurements.** 2.4.0 ends that behaviour.

## Who is affected

| Situation | Impact |
|---|---|
| Account can route to **more than one facility** and sends orders **without `size_system`** | **Breaking — requests now return `400 size_system_ambiguous`** |
| Account is locked to **a single facility** and sends orders **without `size_system`** | Orders still process, but every response carries a `size_system_implicit` warning. **Becomes a hard error in 2.6.** |
| Account already sends `size_system` on every order or line | No change. |

Check your saved order templates as well. Templates are not visible through the API, but they are executed as real orders. A template without `size_system` will hit the same rejection rules as a live request.

## What to do

Add `size_system` at the order level, the line level, or both. Line-level overrides order-level.

### Before (2.3 — fails or warns in 2.4)

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
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### After (2.4+ — explicit, correct)

```json
POST /v2/orders
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

The response will include `resolved_size` on each line:

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

And the header:

```
X-Printf-Size-System: US
```

**Size ladder reference** — chest measurements for `XL`:

| System | chest_cm |
|---|---|
| US | 112 |
| JP | 97 |
| EU | (check your product spec) |

## Error codes

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account can route to multiple facilities and no `size_system` was provided | Add `size_system` to the order or line |
| `size_system_implicit` | — (warning) | Single-facility account; size system was inferred from the facility default | Add `size_system` before 2.6 |

## What happens if you do nothing

- **Multi-facility accounts:** Your orders are **rejected today** with `400 size_system_ambiguous`. You must add `size_system` to resume ordering.
- **Single-facility accounts:** Orders continue to process, but every response carries a `size_system_implicit` warning. **In 2.6, `size_system_implicit` becomes a hard error** and your orders will be rejected with `400`. Add `size_system` before 2.6 ships.
- **Saved order templates:** Templates execute as real orders. Update any template that omits `size_system` — they are subject to the same rules and will fail in the same situations.

