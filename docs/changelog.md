---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Unambiguous sizing

**Released:** 2026-08-21

Size labels are now unambiguous. `size_system` (`US` / `EU` / `JP`) can be set on the order or per line; `fit` is now a per-line field. Response lines and webhook payloads gain `resolved_size` — the canonical chest measurement used for fulfillment. A new `X-Printf-Size-System` response header echoes the system that was applied.

### Behavior changes

| Scenario | Before 2.4.0 | 2.4.0+ |
|---|---|---|
| Bare size, single-facility account | Silently guessed | Resolved, `size_system_implicit` warning |
| Bare size, multi-facility account | Silently guessed (wrong up to 15 cm) | `400 size_system_ambiguous` |
| JP XL chest | 112 cm (wrong) | 97 cm |
| US XL chest | 112 cm | 112 cm |

### New fields

- `size_system` on order and per line — `"US"` / `"EU"` / `"JP"`
- `fit` per line
- `resolved_size` on response lines and webhook payloads: `{ label, system, fit, chest_cm }`

### Migration

Add `size_system` to every order (or per line). Multi-facility accounts **must** do this before 2.6 — bare sizes are a hard error from that version. Single-facility accounts receive `size_system_implicit` warnings starting today.

See the [migration note](https://docs.printf.dev/guides/sizing) for before/after payloads.

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
