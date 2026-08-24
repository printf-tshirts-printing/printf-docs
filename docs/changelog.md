---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

### Breaking change

`size` on order lines is no longer interpreted without an explicit size system. Requests that omit `size_system` on both the line and the order are rejected with `400 size_system_ambiguous` for accounts routable to more than one facility.

### What's new

| Field / header | Scope | Description |
|---|---|---|
| `size_system` | Order body, per line | Size system to apply: `US`, `EU`, or `JP`. Line-level value overrides order-level. |
| `fit` | Per line | Fit designation (e.g. `unisex`, `womens`, `mens`). |
| `resolved_size` | Response lines, webhook payloads | Object confirming how the size was resolved: `label`, `system`, `fit`, `chest_cm`. |
| `X-Printf-Size-System` | Response header | The size system that was ultimately applied to the order. |

### Why it matters

Size ladders differ materially across systems. A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. Silent guessing was causing mis-fulfillment. Explicit disambiguation eliminates the ambiguity at the API layer.

### Warning codes introduced

| Code | Meaning | Becomes error in |
|---|---|---|
| `size_system_implicit` | `size_system` was resolved from the account or facility default, not from the payload. | 2.6 |

### Error codes introduced

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account can route to more than one facility with different size defaults; `size_system` must be explicit. |

### Affected client libraries

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — update to the 2.4.x release of your library to access the new fields.

### Action required

See the [migration note](#migration) for a before/after diff and rollout steps. Accounts routable to more than one facility are already broken — add `size_system` immediately.

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
