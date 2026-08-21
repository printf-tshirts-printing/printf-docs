---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Unambiguous size resolution

`size: "L"` now means the same thing everywhere.

**New fields**

| Field | Scope | Values |
|---|---|---|
| `size_system` | Order root + per line | `US` \| `EU` \| `JP` |
| `fit` | Per line | `unisex`, `mens`, `womens`, etc. |
| `resolved_size` | Response lines + webhook payload | `{ label, system, fit, chest_cm }` |

**New response header:** `X-Printf-Size-System`

**Resolution order for bare size labels:** line `size_system` → order `size_system` → account default → fulfilling facility default.

**Breaking for multi-facility accounts:** bare size labels with no `size_system` set anywhere now return `400 size_system_ambiguous`. Previously the API resolved by guess.

**Deprecation:** single-facility accounts omitting `size_system` receive a `size_system_implicit` warning. This becomes an error in 2.6.

Affected SDKs: printf-js, printf-py, printf-java, printf-go, printf-rb.

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
