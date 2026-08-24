---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

### Breaking change

`size` on order lines must now carry an explicit `size_system` (`US`, `EU`, or `JP`). Bare size labels that cannot be unambiguously resolved are rejected.

### What's new

| Field / header | Where | Notes |
|---|---|---|
| `size_system` | Order body + per line | `US`, `EU`, or `JP` |
| `fit` | Per line | `unisex`, `mens`, `womens` |
| `resolved_size` | Response lines + webhook payloads | `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` | Response header | Effective system used for the order |

### Error and warning codes

- **`400 size_system_ambiguous`** — multi-facility accounts that omit `size_system` and cannot be resolved deterministically.
- **`size_system_implicit` (warning)** — single-facility accounts that omit `size_system`. Becomes a hard error in **2.6**.

### Why ladders differ

A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. Mismatched systems mean garments in the wrong size ship — not a mapping error you can catch in QA.

### Affected SDKs

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — update to the 2.4.x release of your library before deploying.

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
