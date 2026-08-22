---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released 2026-08-21**

### Breaking change

Bare size labels (e.g. `"size": "XL"`) are no longer resolved by guess. Every size must now be paired with a `size_system` value (`US`, `EU`, or `JP`). Accounts that can route to more than one facility and omit `size_system` receive `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning today; this becomes an error in **2.6**.

### New fields

| Field | Scope | Values |
|---|---|---|
| `size_system` | Order body, per line | `US`, `EU`, `JP` |
| `fit` | Per line | `unisex`, and others per garment |
| `resolved_size` | Response lines, webhook payloads | Object: `label`, `system`, `fit`, `chest_cm` |

### New response header

`X-Printf-Size-System` — the system ultimately applied to the order.

### Why this matters

Size ladders differ materially across systems. A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. The previous implicit resolution produced wrong garment sizes for multi-facility accounts silently.

### Affected client libraries

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
