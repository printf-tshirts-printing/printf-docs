---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking)

`size` is no longer resolved without an explicit system. Requests that rely on implicit size resolution now receive a `400 size_system_ambiguous` error (multi-facility accounts) or a `size_system_implicit` warning (single-facility accounts). The warning becomes a hard error in 2.6.

**New fields**

| Location | Field | Type | Notes |
|---|---|---|---|
| Order body | `size_system` | `"US"` / `"EU"` / `"JP"` | Applies to all lines that omit their own `size_system` |
| Line body | `size_system` | `"US"` / `"EU"` / `"JP"` | Overrides the order-level value |
| Line body | `fit` | string | `"unisex"`, `"womens"`, `"mens"` |
| Line response | `resolved_size` | object | `{ label, system, fit, chest_cm }` |
| Response header | `X-Printf-Size-System` | string | Effective system used for the order |

Size ladders differ materially across systems — a JP `XL` chest is 97 cm; a US `XL` is 112 cm. Verify `resolved_size.chest_cm` in staging before sending production orders.

Client libraries updated: **printf-js**, **printf-py**, **printf-java**, **printf-go**, **printf-rb**.

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
