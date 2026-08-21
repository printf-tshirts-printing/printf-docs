---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Unambiguous sizing

**Released 2026-08-21**

`size_system` (`US` / `EU` / `JP`) is now a first-class field on orders and individual lines. Size ladders differ materially — a JP `XL` is 97 cm chest, a US `XL` is 112 cm — so the previous behaviour of resolving bare size labels by guess is replaced with an explicit fallback chain and two new codes.

### What's new

| Feature | Detail |
|---|---|
| `size_system` on order | Applies to all lines that omit their own `size_system` |
| `size_system` per line | Overrides the order-level value for that line only |
| `fit` per line | `unisex`, `womens`, `mens` |
| `resolved_size` on response lines | Object: `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` response header | System that resolved for the order |
| `size_system_ambiguous` (400) | Multi-facility accounts sending no `size_system` |
| `size_system_implicit` warning | Single-facility accounts sending no `size_system`; becomes error in 2.6 |

### Who must act before 2.6

Any integration that sends bare size labels (`"size": "L"`) without an explicit `size_system` on the order or per line. Check saved order templates — they inherit the old ambiguous behaviour and are not updated automatically.

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
