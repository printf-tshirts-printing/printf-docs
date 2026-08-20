---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-20
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
---

# Bulk orders and templates

order-api 2.4.0 adds required size-system context to every order. **Saved order templates are affected even if you never touch them** — read the template section below before your next batch run.

## Bulk requests

Each order in a bulk batch is an independent `POST /v2/orders` call. The size-system fields apply to every one:

| Field | Level | Required |
|---|---|---|
| `size_system` | Order or per line | Required for multi-facility accounts; strongly recommended for all |
| `fit` | Per line | Optional; defaults to `unisex` |

### Before (2.3.x)

```json
{
  "accountId": "acct_stackfest",
  "destination": { "name": "StackFest Ops", "line1": "410 Congress Ave", "city": "Austin", "region": "TX", "postalCode": "78701", "countryCode": "US" },
  "lines": [
    { "designId": "dsn_7fa91c", "size": "XL", "quantity": 250, "garmentSku": "tee-classic-black" }
  ]
}
```

### After (2.4.0+)

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "destination": { "name": "StackFest Ops", "line1": "410 Congress Ave", "city": "Austin", "region": "TX", "postalCode": "78701", "countryCode": "US" },
  "lines": [
    { "designId": "dsn_7fa91c", "size": "XL", "size_system": "US", "fit": "unisex", "quantity": 250, "garmentSku": "tee-classic-black" }
  ]
}
```

## `400 size_system_ambiguous` in batch runs

If your account routes to more than one facility (e.g. `fac-atx` and `fac-ber`) and any order in the batch omits `size_system`, that order is rejected with `400 size_system_ambiguous`. Other orders in the batch are unaffected. Add `size_system` at the order level to cover all lines, or per line to override individually.

## Saved order templates — action required

Saved templates do not appear in the API response for `GET /v2/orders`. They are stored server-side and replayed verbatim when triggered. **A template saved before 2.4.0 will not contain `size_system`.**

- **Single-facility accounts:** the template will trigger a `size_system_implicit` warning on every order it creates. This becomes a `400` in 2.6. Update the template before 2.6.
- **Multi-facility accounts:** the template will be rejected with `400 size_system_ambiguous` immediately.

To update a saved template, re-submit it via `POST /v2/orders` with `size_system` and `fit` added, then save the updated version. There is no PATCH endpoint for templates.

## SDK status

`printf-js` and `printf-py` do not yet support `size_system` or `fit`. Use raw HTTP for new bulk payloads until SDK updates ship. Existing SDK-generated bulk payloads that omit `size_system` will fail for multi-facility accounts and warn for single-facility accounts.
