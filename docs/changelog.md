---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

`size` now requires an explicit size system. The bare label alone is rejected or warned depending on your account configuration.

**New fields**

| Field | Scope | Values |
|---|---|---|
| `size_system` | Order body, line item | `US`, `EU`, `JP` |
| `fit` | Line item | `unisex`, and any per-garment values |
| `resolved_size` | Response line, webhook payload | Object: `label`, `system`, `fit`, `chest_cm` |

**New response header**: `X-Printf-Size-System` reflects the system used to resolve sizes for the request.

**Breaking change**: Accounts that can route to more than one facility and omit `size_system` receive `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning today; this becomes an error in 2.6.

**Affected client libraries**: printf-js, printf-py, printf-java, printf-go, printf-rb

**Size ladder differences** (chest measurement at XL)

| System | XL chest |
|---|---|
| US | 112 cm |
| JP | 97 cm |
| EU | varies by garment |

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
