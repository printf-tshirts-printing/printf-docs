---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2026-08-21 — Orders API 2.4.0

**Sizing and fit: `size` is no longer ambiguous.**

- `POST /v2/orders` now accepts `size_system` (`US` / `EU` / `JP`) at the order level and per line; each line also accepts `fit`.
- Responses and webhook payloads include `resolved_size` per line (`label`, `system`, `fit`, `chest_cm`). A new `X-Printf-Size-System` response header echoes the resolved system.
- Accounts that can route to more than one facility **must** send `size_system` explicitly. Omitting it now returns `400 size_system_ambiguous` instead of silently resolving to a facility guess.
- Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning today. This becomes a hard error in **2.6**.

Saved order templates carry no explicit `size_system`. [Update them before 2.6](/guides/sizing-and-fit#migrating-saved-templates).

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
