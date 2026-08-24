---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## Size disambiguation — order-api 2.4.0

**Released 2026-08-24**

### Breaking change

`size` on order lines must now be accompanied by a `size_system` value (`US`, `EU`, or `JP`). Bare size labels are no longer silently resolved by guess.

- Accounts that can route to more than one facility are rejected immediately with `400 size_system_ambiguous`.
- Single-facility accounts receive a `size_system_implicit` warning for now; this becomes an error in 2.6.

### New fields

| Field | Scope | Description |
|---|---|---|
| `size_system` | Order body, line item | Declares the size ladder: `US`, `EU`, or `JP` |
| `fit` | Line item | Garment fit: `unisex`, `mens`, `womens` |
| `resolved_size` | Response line, webhook payload | The canonical size after resolution: label, system, fit, chest_cm |

### New response header

`X-Printf-Size-System` — the system that was applied when resolving sizes on the order.

### Why this matters

Size ladders differ materially across systems. A JP `XL` is 97 cm chest; a US `XL` is 112 cm. Silent guessing produced the wrong garment for cross-region orders.

### Affected SDKs

printf-js, printf-py, printf-java, printf-go, printf-rb — update to the 2.4.x release of your library.

### Migration

See the full migration note below. The short version: add `size_system` and `fit` to every order line before 2.6 ships.

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
