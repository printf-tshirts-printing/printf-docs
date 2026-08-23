---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking)

**Released 2026-08-23**

Size labels now require an explicit system. Bare sizes are still accepted during a transition window but the behaviour has changed.

### What's new

| Field | Scope | Notes |
|---|---|---|
| `size_system` | Order + line | `US` / `EU` / `JP`. Line value overrides order value. |
| `fit` | Line | `unisex`, `womens`, `mens`. |
| `resolved_size` | Response line + webhook | `{ "label", "system", "fit", "chest_cm" }` — the exact size applied at the facility. |
| `X-Printf-Size-System` | Response header | The system that was resolved for the request. |

### Breaking change — `size_system_ambiguous` (400)

Accounts routable to **more than one facility** that send a bare `size` with no `size_system` now receive `400 size_system_ambiguous`. Previously the API guessed. Affected accounts: StackFest, Cloud Native Rodeo, ObservaCon, KubeSummit, ShipItConf.

### Deprecation — `size_system_implicit` warning

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in the response. This warning **becomes an error in 2.6**.

### Size ladder note

Ladders differ materially across systems (JP XL = 97 cm chest, US XL = 112 cm chest). Verify your size mappings before going live.

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
