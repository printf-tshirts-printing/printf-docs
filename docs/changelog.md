---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking)

**Released:** 2026-08-24

### Breaking changes

- `size` on order lines is no longer resolved without an explicit size system. Requests that omit `size_system` on both the line and the order body and that route to more than one facility are rejected with **`400 size_system_ambiguous`**.

### New fields

| Location | Field | Type | Notes |
|---|---|---|---|
| Order body | `size_system` | `US` \| `EU` \| `JP` | Default for all lines |
| Order line | `size_system` | `US` \| `EU` \| `JP` | Overrides order-level value |
| Order line | `fit` | string | e.g. `unisex`, `womens`, `mens` |
| Response line | `resolved_size` | object | `{ label, system, fit, chest_cm }` |

### New response header

`X-Printf-Size-System` — the effective size system used to resolve the order.

### Deprecation warning

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in 2.4. This warning becomes a hard error in **2.6**.

### SDK versions shipping with this release

| Library | Version |
|---|---|
| printf-js | 2.4.0 |
| printf-py | 2.4.0 |
| printf-java | 2.4.0 |
| printf-go | 2.4.0 |
| printf-rb | 2.4.0 |

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
