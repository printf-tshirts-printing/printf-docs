---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking change)

**Released:** 2026-08-22

`size` on order lines now requires an explicit size system. Bare size labels that cannot be resolved deterministically are rejected.

### What's new

| Feature | Detail |
|---|---|
| `size_system` on order | `US` / `EU` / `JP`. Applies to all lines that omit their own `size_system`. |
| `size_system` per line | Overrides the order-level value for that line. |
| `fit` per line | `unisex`, `womens`, `mens`. Affects chest-measurement ladder selection. |
| `resolved_size` in responses and webhooks | `{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }` |
| `X-Printf-Size-System` response header | The system actually applied to the order. |

### Breaking change

Accounts that route to **more than one facility** and supply no `size_system` are rejected with `400 size_system_ambiguous`. Previously, the fulfilling facility's default was applied silently — a JP `XL` (97 cm) and a US `XL` (112 cm) are not the same garment.

### Deprecation

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in 2.4. This warning becomes a hard error in **2.6**.

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
