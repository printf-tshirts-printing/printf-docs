---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## Size disambiguation — order-api 2.4.0

Adds explicit size-system support to orders and lines. **Breaking change for multi-facility accounts.**

### What's new

| Field / header | Where | Values |
|---|---|---|
| `size_system` | Order body, per-line body | `US` \| `EU` \| `JP` |
| `fit` | Per-line body | e.g. `unisex`, `womens`, `mens` |
| `resolved_size` | Response lines, webhook payloads | `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` | Response header | The system that resolved the order |

### Breaking change

Bare size labels (e.g. `"size": "XL"` with no `size_system`) are no longer silently resolved by guess for accounts that can route to more than one facility. Those requests now return `400 size_system_ambiguous`.

Single-facility accounts continue to work but receive a `size_system_implicit` warning starting in 2.4. That warning becomes a hard error in **2.6**.

### Why it matters

Size ladders differ materially across systems: JP `XL` = 97 cm chest; US `XL` = 112 cm. Silent resolution was producing wrong garments for cross-facility accounts.

### Affected clients

printfjs, printf-py, printf-java, printf-rb, printf-go

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
