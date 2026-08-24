---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change** for accounts that route to more than one facility.

### What's new

- **`size_system`** field on the order root and per line item (`US` / `EU` / `JP`). Controls which size ladder is used to resolve garment dimensions.
- **`fit`** per line item (`unisex`, `womens`, `mens`). Replaces the previously implicit unisex assumption.
- **`resolved_size`** on all order response bodies and webhook payloads. Each line now echoes back `{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }` so you can verify what the fulfillment system received.
- **`X-Printf-Size-System`** response header. Contains the system that was ultimately applied — useful for logging and debugging.

### Breaking change

Bare size labels (e.g. `"size": "XL"` with no `size_system`) are no longer silently resolved by guess for accounts that can route to more than one facility. These requests now return `400 size_system_ambiguous`.

Single-facility accounts continue to work but receive a `size_system_implicit` warning in the response. This warning becomes an error in **2.6**.

### Size ladder differences (selected)

| Label | US chest (cm) | EU chest (cm) | JP chest (cm) |
|-------|--------------|--------------|------------------|
| M | 99 | 96 | 88 |
| L | 106 | 101 | 92 |
| XL | 112 | 106 | 97 |

A JP `XL` is 15 cm narrower across the chest than a US `XL`. Do not assume portability between systems.

### Affected clients

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — all updated to 2.4.0. Pass `size_system` through the SDK's order builder; existing code that omits it will surface `size_system_ambiguous` errors or `size_system_implicit` warnings at runtime.

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
