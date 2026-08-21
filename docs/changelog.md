---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size system, fit, and resolved_size

**Released 2026-08-21**

Size labels are no longer ambiguous. A JP `XL` (97 cm chest) is not a US `XL` (112 cm chest), and the API now enforces the distinction.

### What's new

| Field / Header | Where | Notes |
|---|---|---|
| `size_system` | `POST /v2/orders` (order level) | `US` \| `EU` \| `JP` |
| `size_system` | `POST /v2/orders` line items | Overrides order-level value for that line |
| `fit` | `POST /v2/orders` line items | e.g. `unisex`, `womens`, `mens` |
| `resolved_size` | Order response + webhook payloads | `{ label, system, fit, chest_cm }` |
| `X-Printf-Size-System` | All order responses | The system used for the order |

### New error and warning codes

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` supplied and account routes to more than one facility. Request rejected. |
| `size_system_implicit` | warning | No `size_system` supplied on a single-facility account. Request succeeds now; becomes an error in 2.6. |

### Who is affected

Any account sending bare size labels (e.g. `"size": "L"`) without `size_system` on the order or per line. Multi-facility accounts get a hard 400 immediately. Single-facility accounts get a deprecation warning until 2.6.

**Action required before 2.6:** add `size_system` to every order and every saved template.

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
