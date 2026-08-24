---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change** for multi-facility accounts.

Adds explicit size-system support to orders and line items. `size` is no longer interpreted without a declared system — bare labels now resolve through a fallback chain (line → order → account → facility default), and accounts that route to more than one facility are rejected with `400 size_system_ambiguous` rather than guessed.

### New fields

| Field | Scope | Values |
|---|---|---|
| `size_system` | Order body, per line | `US`, `EU`, `JP` |
| `fit` | Per line | e.g. `unisex`, `mens`, `womens` |
| `resolved_size` | Response lines, webhook payloads | `{ label, system, fit, chest_cm }` |

### New response header

`X-Printf-Size-System` — echoes the size system that was applied to the order.

### Why this matters

Size ladders differ materially across systems. JP `XL` = 97 cm chest; US `XL` = 112 cm chest. Prior to 2.4.0 ambiguous bare labels resolved silently by facility default, producing wrong garments with no signal in the response.

### Affected clients

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb`

### Migration

See the full migration note. Single-facility accounts receive a `size_system_implicit` warning now; it becomes an error in 2.6.

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
