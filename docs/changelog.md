---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2026-08-25 — Orders API 2.4.0

**Sizing and fit — breaking change for multi-facility accounts.**

- `POST /v2/orders` accepts `size_system` (`US`, `EU`, `JP`) at the order level and per line, and `fit` per line.
- Response lines now include `resolved_size`: `{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }`.
- Accounts routing to more than one facility now receive `400 size_system_ambiguous` instead of a silent facility-default guess. If you call `POST /v2/orders` without `size_system` and your account covers multiple facilities, your requests will fail.
- `printf-js`, `printf-py`, `printf-java`, `printf-go`, and `printf-rb` all surface `size_system` and `fit` in this release.

Saved order templates created before this release carry no explicit `size_system` and will trigger `size_system_ambiguous` on multi-facility accounts. [See what to change](/guides/sizing#migrating-saved-templates).

Full migration note: [Sizing and fit → Migrating to 2.4.0](/guides/sizing#migrating-to-240). `size_system_ambiguous` becomes a hard error for all accounts in 2.5.0.

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
