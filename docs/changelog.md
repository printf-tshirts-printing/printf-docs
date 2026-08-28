---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2026-08-28 — Orders API 2.4.0

**Size disambiguation — breaking change for multi-facility accounts.**

- **Breaking:** Accounts that can route to more than one facility now receive `400 size_system_ambiguous` instead of a facility-default guess. Add `size_system` (`US`, `EU`, `JP`) to the order or each line to resolve.
- `POST /v2/orders` accepts `size_system` at the order level and per line, and `fit` per line.
- Responses and webhook payloads carry `resolved_size` per line; the new `X-Printf-Size-System` response header echoes the resolved system.
- Single-facility accounts receive a `size_system_implicit` warning today; this becomes an error in 2.6. Explicit `size_system` silences it.
- Saved order templates carry no `size_system` and will resolve against facility defaults until updated — or break at 2.6. [What to change](/guides/sizing#migrating-saved-templates).

Ladder note: JP `XL` = 97 cm chest; US `XL` = 112 cm. Wrong system, wrong garment.

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
