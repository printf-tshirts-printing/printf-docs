---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size system disambiguation

**Released:** 2026-08-21

### What changed

- **`size_system` field** (`US` / `EU` / `JP`) is now accepted on the order root and per line item. Per-line value takes precedence over the order-level default.
- **`fit` field** is now accepted per line item (`unisex`, `mens`, `womens`).
- **`resolved_size` object** is now returned on every response line and in webhook payloads: `{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }`.
- **`X-Printf-Size-System` response header** indicates the resolved system for the whole order.
- Bare size labels now resolve via a priority chain: line → order → account → facility default.
- Multi-facility accounts that omit `size_system` entirely now receive `400 size_system_ambiguous` instead of a silent guess.
- Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning. This becomes an error in **2.6.0**.

### Why it matters

A JP `XL` chest is 97 cm. A US `XL` chest is 112 cm. The previous behaviour silently resolved to whichever facility happened to fulfil the order, producing wrong-size garments with no error signal.

### Affected clients

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb`

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
