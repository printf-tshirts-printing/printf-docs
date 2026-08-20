---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — 2026-08-20

### Added
- `size_system` field on orders and per line item (`US` / `EU` / `JP`). Controls which measurement ladder resolves bare size labels.
- `fit` field per line item (`unisex`, and others). Feeds into size resolution.
- `resolved_size` object on response line items and webhook payloads: `{ "label", "system", "fit", "chest_cm" }`.
- `X-Printf-Size-System` response header indicating which size system was applied to the order.

### Changed
- Bare size labels (e.g. `"XL"`) now resolve via a priority chain: line → order → account → fulfilling facility default. Multi-facility accounts that rely on the last step receive `400 size_system_ambiguous`.

### Deprecated
- Omitting `size_system` on single-facility accounts produces a `size_system_implicit` warning. This becomes a hard error in 2.6.0.

### Notes
- JP `XL` = 97 cm chest; US `XL` = 112 cm chest. Ladders are not interchangeable.
- All five official SDK clients (`printf-js`, `printf-py`, `printf-go`, `printf-java`, `printf-rb`) report `supportsSizeSystem=false` and do not yet model the new fields. SDK releases are tracked separately.

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
