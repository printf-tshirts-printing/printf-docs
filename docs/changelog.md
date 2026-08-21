---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — 2026-08-21

### Size system disambiguation

Size labels are no longer ambiguous. Every order can now carry an explicit size system so that `XL` means what you mean, not what the facility guesses.

**New fields**

| Location | Field | Type | Notes |
|---|---|---|---|
| Order root | `size_system` | `"US"` \| `"EU"` \| `"JP"` | Applies to all lines that omit their own |
| Line item | `size_system` | `"US"` \| `"EU"` \| `"JP"` | Overrides the order-level value |
| Line item | `fit` | string | e.g. `"unisex"`, `"womens"` |
| Response / webhook line | `resolved_size` | object | `{ label, system, fit, chest_cm }` |

**New response header:** `X-Printf-Size-System` — the system that was applied to the order.

**Resolution order** when `size_system` is omitted: line → order → account default → fulfilling facility default.

**Accounts that can route to more than one facility** can no longer rely on the facility fallback. Omitting `size_system` on such accounts now returns `400 size_system_ambiguous`.

**Single-facility accounts** receive a `size_system_implicit` warning today. This warning becomes an error in 2.6.

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
