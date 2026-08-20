---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — 2026-08-20

### Added
- `size_system` field on the order root and per line (`US` | `EU` | `JP`). Controls which size ladder resolves bare labels like `L` or `XL`.
- `fit` field per line (`unisex` | `mens` | `womens` | `youth`). Required when `size_system` is present on the line.
- `resolved_size` object on every response line and webhook payload: `{ "label", "system", "fit", "chest_cm" }`.
- `X-Printf-Size-System` response header on `POST /v2/orders` and `GET /v2/orders/{id}`. Value is the effective size system used to resolve the order.

### Changed
- Bare size labels (e.g. `"XL"`) now resolve using the nearest explicit `size_system`: line → order → account → fulfilling facility default. Accounts that can route to more than one facility and omit `size_system` receive `400 size_system_ambiguous` instead of a silently guessed resolution.

### Deprecated
- Omitting `size_system` on single-facility accounts now emits warning `size_system_implicit`. **This warning becomes a hard error in 2.6.**

### Notes
- Ladder semantics differ materially across systems: JP `XL` = 97 cm chest; US `XL` = 112 cm chest. Review all saved order templates before the next order run.

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
