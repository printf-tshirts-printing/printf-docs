---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change** for multi-facility accounts.

The `size` field now requires an explicit size system. A bare label like `XL` is no longer resolved by guess when an account can route to more than one fulfillment facility.

### New fields

| Location | Field | Values |
|---|---|---|
| Order body | `size_system` | `US` / `EU` / `JP` |
| Line item body | `size_system` | `US` / `EU` / `JP` |
| Line item body | `fit` | `unisex` / `mens` / `womens` |
| Line item response | `resolved_size` | `{ label, system, fit, chest_cm }` |
| Response header | `X-Printf-Size-System` | Resolved system identifier |

### New error and warning codes

- `400 size_system_ambiguous` — multi-facility accounts sending a bare size label with no resolvable system.
- `size_system_implicit` warning — single-facility accounts relying on the facility default. Becomes a hard error in 2.6.

### Ladder differences

Size ladders differ materially across systems. A JP `XL` is 97 cm chest; a US `XL` is 112 cm. Verify garment specs against the system you declare.

### Affected client libraries

printf-js · printf-py · printf-java · printf-go · printf-rb

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
