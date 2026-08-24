---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation (breaking)

**Released:** 2026-08-24

### What changed

`size` on order lines is no longer interpreted without an explicit size system. This removes a class of silent fulfillment errors where a JP `XL` (97 cm chest) was shipped when the customer intended a US `XL` (112 cm).

### New fields

| Field | Scope | Type | Values |
|---|---|---|---|
| `size_system` | Order body | string | `US` \| `EU` \| `JP` |
| `size_system` | Line item | string | `US` \| `EU` \| `JP` |
| `fit` | Line item | string | e.g. `unisex`, `women`, `men` |
| `resolved_size` | Response / webhook | object | `{ label, system, fit, chest_cm }` |

New response header: `X-Printf-Size-System`.

### Breaking change

Accounts that can route to **more than one fulfilling facility** and omit `size_system` on every level (line → order → account) now receive `400 size_system_ambiguous`. Previously the API guessed; it no longer does.

Single-facility accounts receive a `size_system_implicit` warning for the same omission. That warning becomes an error in **2.6**.

### Affected clients

printf-js, printf-py, printf-java, printf-go, printf-rb — update to the 2.4.x patch of each SDK to gain typed `size_system`, `fit`, and `resolved_size` fields.

### Action required

See the [migration note](https://docs.printf.dev/guides/sizing) for a before/after diff and a checklist for saved order templates.

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
