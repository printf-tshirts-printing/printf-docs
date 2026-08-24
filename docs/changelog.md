---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation (breaking)

**Released 2026-08-24**

`size` now requires an explicit size system. Bare labels that previously resolved by guess are rejected or warned.

### New fields

| Field | Scope | Type | Notes |
|---|---|---|---|
| `size_system` | Order body, line item | `US` \| `EU` \| `JP` | Sets the size ladder for interpretation |
| `fit` | Line item | string | e.g. `unisex`, `fitted`, `relaxed` |
| `resolved_size` | Response line, webhook payload | object | `{ label, system, fit, chest_cm }` |

### New response header

`X-Printf-Size-System: US` — the system used to resolve all lines in the request.

### Breaking change

Accounts that can route to **more than one facility** and send a bare `size` with no `size_system` receive `400 size_system_ambiguous`. Previously these requests resolved by guess.

Single-facility accounts receive a `size_system_implicit` warning today. That warning becomes an error in **2.6**.

### Why it matters

A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. Silent misresolution shipped wrong-sized garments.

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
