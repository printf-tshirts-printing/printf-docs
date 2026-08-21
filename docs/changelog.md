---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Unambiguous sizing

**Released 2026-08-21**

Size labels such as `"L"` or `"XL"` now resolve deterministically. This release adds a size-system field at the order and line level, a fit field per line, and a `resolved_size` object on every response and webhook payload.

### What's new

| Field / header | Scope | Values |
|---|---|---|
| `size_system` | Order body, line item | `US`, `EU`, `JP` |
| `fit` | Line item | e.g. `unisex`, `womens`, `mens` |
| `resolved_size` | Response line, webhook payload | `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` | Response header | Resolved system for the order |

### Resolution order

When `size_system` is omitted, the API resolves it: line → order → account → fulfilling facility default. Accounts that can route to more than one facility are rejected with `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning today; this becomes an error in **2.6**.

### Breaking risk

- JP `XL` = 97 cm chest. US `XL` = 112 cm chest. Wrong system → wrong garment.
- Multi-facility accounts that omit `size_system` will receive `400 size_system_ambiguous` immediately.
- All other accounts that omit `size_system` will receive `size_system_implicit` warnings until 2.6.

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
