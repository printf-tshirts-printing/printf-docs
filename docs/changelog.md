---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change** for multi-facility accounts.

### What's new

- **`size_system`** field on order root and per line (`US` / `EU` / `JP`). Controls which size ladder is used for every line.
- **`fit`** field per line (`unisex`, `mens`, `womens`). Previously implicit.
- **`resolved_size`** on response lines and webhook payloads: `{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }`.
- **`X-Printf-Size-System`** response header: the system actually applied to the order.

### Breaking change

`size` is no longer resolved by guessing a size system. Bare size labels now follow a resolution chain: line → order → account → fulfilling facility default. Accounts that can route to **more than one facility** are rejected with `400 size_system_ambiguous` — the fallback guess is gone.

Single-facility accounts receive a `size_system_implicit` warning today; this becomes an error in **2.6**.

### Size ladder note

Ladders differ materially between systems. A JP `XL` is 97 cm chest; a US `XL` is 112 cm. Resolve this explicitly — do not rely on facility defaults for international garments.

### Affected client libraries

`printf-js`, `printf-py`, `printf-java`, `printf-rb`, `printf-go`

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
