---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

**Breaking change** — `size` values without an explicit size system are no longer silently resolved.

### What's new

| Feature | Details |
|---|---|
| `size_system` on order and per line | Accepts `US`, `EU`, or `JP`. |
| `fit` per line | Accepted on `lines[]` items. |
| `resolved_size` on responses and webhooks | Returns `{ label, system, fit, chest_cm }` per line. |
| `X-Printf-Size-System` response header | Reflects the system used to resolve sizes for the request. |

### Breaking behavior

Bare size labels (e.g. `"XL"` with no `size_system`) previously resolved by internal guess. As of 2.4.0:

- Accounts routable to **more than one facility** receive `400 size_system_ambiguous`.
- Accounts routable to **exactly one facility** receive a `size_system_implicit` warning in the response. This warning becomes an error (`400`) in 2.6.0.

### Why it matters

Ladders differ materially. A JP `XL` is 97 cm chest; a US `XL` is 112 cm. Silent resolution was producing wrong garment sizes.

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
