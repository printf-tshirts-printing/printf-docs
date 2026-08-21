---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size system disambiguation

**Released:** 2026-08-21

`size: "L"` is no longer ambiguous. This release adds explicit size system support across the order API.

### What's new

| Field / Header | Where | Description |
|---|---|---|
| `size_system` | Request body (order-level and per line) | `US`, `EU`, or `JP`. Line value overrides order value. |
| `fit` | Request body (per line) | Garment fit, e.g. `unisex`. |
| `resolved_size` | Response lines and webhook payloads | `{ "label", "system", "fit", "chest_cm" }` |
| `X-Printf-Size-System` | Response header | The size system applied to the order. |

### New error codes

| Code | Type | Meaning |
|---|---|---|
| `size_system_ambiguous` | `400` | Multi-facility accounts that omit `size_system`; rejected instead of guessed. |
| `size_system_implicit` | Warning | Single-facility accounts that omit `size_system`; becomes an error in 2.6. |

### Resolution ladder

Bare size labels resolve: line → order → account → fulfilling facility default.

Ladders differ materially: JP `XL` = 97 cm chest, US `XL` = 112 cm chest.

### Action required

All five SDK clients (`printf-js`, `printf-py`, `printf-go`, `printf-java`, `printf-rb`) do not yet model `size_system`, `fit`, or `resolved_size`. Updates are in progress. Keynote customers routing to multiple facilities will see `400 size_system_ambiguous` starting today — add `size_system` to every order and line to resolve.

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
