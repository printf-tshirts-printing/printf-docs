---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Unambiguous sizing

**Released 2026-08-21**

`size: "L"` now means one thing. 2.4.0 adds explicit size-system fields so every order resolves to a single physical measurement.

### What's new

| Field | Where | Values |
|---|---|---|
| `size_system` | Order root + per line | `US`, `EU`, `JP` |
| `fit` | Per line | e.g. `unisex` |
| `resolved_size` | Response lines + webhook payloads | `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` | Response header | Effective system for the order |

### Resolution chain

When `size_system` is omitted, the API walks: **line → order → account → fulfilling facility default**. The facility-default step depends on routing, so:

- **Multi-facility accounts** that omit `size_system` receive `400 size_system_ambiguous`.
- **Single-facility accounts** that omit `size_system` receive a `size_system_implicit` warning — this becomes an error in **2.6**.

### Why this matters

A JP `XL` chest is **97 cm**. A US `XL` chest is **112 cm**. Getting this wrong ships the wrong garment.

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
