---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size system resolution (2026-08-21)

**Size labels are no longer ambiguous.** This release adds explicit size system support to the Orders API.

### New fields

| Field | Where | Type | Description |
|---|---|---|---|
| `size_system` | Order body, line item | `US` \| `EU` \| `JP` | Declares the size ladder for the order or individual line |
| `fit` | Line item | string | e.g. `unisex`, `womens`, `mens` |
| `resolved_size` | Response line, webhook payload | object | `{ label, system, fit, chest_cm }` |

### New response header

`X-Printf-Size-System` — the system that was applied to resolve bare size labels on the request.

### Resolution order (bare size labels)

Line → Order → Account → Fulfilling facility default.

The facility-default fallback depends on routing, not the payload:
- **Multi-facility accounts** that hit the facility-default step are rejected: `400 size_system_ambiguous`
- **Single-facility accounts** receive a `size_system_implicit` warning. This becomes an error in **2.6**.

### Why it matters

A JP `XL` chest is 97 cm. A US `XL` chest is 112 cm. Silent misresolution ships the wrong garment.

### Affected client libraries

printf-js, printf-py, printf-java, printf-go, printf-rb — update to the latest minor for typed `size_system` and `fit` fields and `resolved_size` on response models.

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
