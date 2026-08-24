---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

### Breaking change

`size` on order lines is no longer interpreted without a known size system. Requests that previously resolved a bare size label by guessing the fulfilling facility's default will now be rejected with `400 size_system_ambiguous` if the account can route to more than one facility.

### What's new

| Field / header | Scope | Description |
|---|---|---|
| `size_system` | Order + line | `US`, `EU`, or `JP`. Line value takes precedence over order value. |
| `fit` | Line | Per-line fit qualifier (e.g. `unisex`). |
| `resolved_size` | Response + webhook | Object on each response line: `{ label, system, fit, chest_cm }`. |
| `X-Printf-Size-System` | Response header | The size system that was applied to the order. |

### Warning codes introduced

- `size_system_implicit` — emitted when a single-facility account omits `size_system` and the facility default is used. **Becomes a hard error in 2.6.**

### Error codes introduced

- `size_system_ambiguous` — `400` returned when a multi-facility account omits `size_system` and routing cannot determine a single default.

### Why size ladders matter

A JP `XL` is **97 cm** chest. A US `XL` is **112 cm** chest. Omitting `size_system` on a cross-border order silently ships the wrong garment. `resolved_size.chest_cm` in every response lets you confirm the ladder that was applied.

### Affected client libraries

`printf-js`, `printf-py`, `printf-java`, `printf-rb`, `printf-go` — each library's next patch release adds typed `SizeSystem` and `Fit` enums and surfaces `resolved_size` on line response objects.

### Keynote accounts with active orders

StackFest, Cloud Native Rodeo, ObservaCon, KubeSummit, ShipItConf — account managers have been notified. Migration deadline is the 2.6 release window.

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
