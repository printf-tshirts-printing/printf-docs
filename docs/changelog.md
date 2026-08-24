---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking)

**Released:** 2026-08-24

`size` on order lines now requires an explicit size system. Pass `size_system` (`US` / `EU` / `JP`) at the line, order, or account level. Bare labels that could route to more than one facility are rejected with `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning — this becomes an error in 2.6.

**New fields**

| Field | Scope | Values |
|---|---|---|
| `size_system` | order body + per line | `US`, `EU`, `JP` |
| `fit` | per line | `unisex`, and others |
| `resolved_size` | response lines + webhook | `{ label, system, fit, chest_cm }` |

**New response header:** `X-Printf-Size-System`

**Client libraries updated:** printf-js, printf-py, printf-java, printf-go, printf-rb

See the [migration note](#migration) and the [Sizing and fit guide](docs/guides/sizing.md).

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
