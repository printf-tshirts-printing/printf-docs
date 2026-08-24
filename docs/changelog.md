---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation (breaking)

**Released:** 2026-08-24

### What changed

Size labels are no longer interpreted without an explicit size system. This is a **breaking change** for any caller that sends a bare `size` without `size_system`.

### New fields

| Location | Field | Type | Values |
|---|---|---|---|
| Request (order) | `size_system` | string | `US` \| `EU` \| `JP` |
| Request (line) | `size_system` | string | `US` \| `EU` \| `JP` |
| Request (line) | `fit` | string | e.g. `unisex`, `womens`, `mens` |
| Response / webhook (line) | `resolved_size` | object | `{ label, system, fit, chest_cm }` |
| Response header | `X-Printf-Size-System` | string | The system used to resolve all lines |

### New error and warning codes

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` provided and account can route to more than one facility |
| `size_system_implicit` | — (warning) | No `size_system` provided; resolved from single-facility default. Becomes an error in 2.6 |

### Why this matters

Size ladders differ materially between systems. A JP `XL` chest is **97 cm**; a US `XL` chest is **112 cm**. Silent mis-resolution was shipping garments in the wrong size.

### Affected SDKs

`printf-js` · `printf-py` · `printf-java` · `printf-go` · `printf-rb` — update to the 2.4.x release of your client library.

## 2026-07-28 — Orders API 2.3.6
Designs above 40 MB now fail fast with `art_too_large` instead of timing out.

## 2026-07-09 — Orders API 2.3.4
Order responses carry `warnings[]`. Deprecations land there one minor before
they become errors.

## 2026-06-15 — Orders API 2.3.0
**Multi-facility routing.** An account can now be served by more than one
facility. Orders route per-destination based on stock and capacity. Pin a
facility with `facilityId` if you need the old single-site behaviour.

## 2026-04-30 — Orders API 2.2.0
Sandbox environment at `api.sandbox.printf.dev`.
