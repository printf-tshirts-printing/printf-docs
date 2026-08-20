---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-20
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-go
  - printf-java
  - printf-rb
---

# Sizing and fit

order-api 2.4.0 replaces the single bare size label with an explicit size system. This page explains how size resolution works, what the ladders look like, and what you must change before 2.6.

## Size systems

| `size_system` value | Standard | Example: `XL` chest |
|---|---|---|
| `US` | US/Canada unisex | 112 cm |
| `EU` | European | 108 cm |
| `JP` | Japanese | 97 cm |

A JP `XL` and a US `XL` differ by 15 cm. Always set `size_system` explicitly.

## Where to set `size_system`

`size_system` can appear in three places, resolved from most-specific to least:

1. **Per line** — `lines[].size_system`
2. **Order level** — top-level `size_system`
3. **Fulfilling facility default** — resolved by routing, only used when neither of the above is set

The fallback to facility default only works for **single-facility accounts**. If your account can route to more than one facility (Austin `fac-atx`, Berlin `fac-ber`, Osaka `fac-osa`) and you omit `size_system` at every level, the request is rejected with `400 size_system_ambiguous`.

## `fit`

Set `fit` per line. Accepted values: `unisex`, `women`, `men`. Fit affects cut but not the size ladder.

## `resolved_size` in responses

Every response line and webhook payload now includes a `resolved_size` object:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` for display and QA. The `X-Printf-Size-System` response header echoes the system resolved for the order.

## Warnings and errors

| Code | Introduced | Becomes error |
|---|---|---|
| `size_system_ambiguous` | 2.4.0 | Already a `400` |
| `size_system_implicit` | 2.4.0 | 2.6 |

`size_system_implicit` is issued when a single-facility account omits `size_system` and the API falls back to the facility default. **Update your requests before 2.6 or they will be rejected.**

## SDK status

All five SDKs have `supportsSizeSystem=false` as of 2026-08-20. Pass `size_system` and `fit` via the raw HTTP client until SDK updates ship.

| SDK | Version | `supportsSizeSystem` |
|---|---|---|
| printf-js | 2.3.4 | `false` |
| printf-py | 2.3.2 | `false` |
| printf-go | 2.3.1 | `false` |
| printf-java | 3.2.1 | `false` |
| printf-rb | 2.3.0 | `false` |

Watch each SDK's release notes for the version that sets `supportsSizeSystem=true`.
