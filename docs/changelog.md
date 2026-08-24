---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Breaking change** — bare size labels without an explicit size system are no longer silently resolved.

### What's new

- `size_system` field (`US` / `EU` / `JP`) accepted on the order root and per line item. Per-line value overrides the order-level value.
- `fit` field per line item (`unisex`, `mens`, `womens`).
- `resolved_size` object on every response line and webhook payload: `{ "label", "system", "fit", "chest_cm" }`.
- `X-Printf-Size-System` response header indicating the system used to resolve sizes.
- New error code `400 size_system_ambiguous` — returned when a bare size label reaches routing and more than one facility could fulfil the order with different size ladders.
- New warning code `size_system_implicit` — returned when a bare size label is resolved by a single-facility account's facility default. Becomes a hard error in **2.6.0**.

### Breaking change detail

Size ladders differ materially between systems. A JP `XL` has a 97 cm chest; a US `XL` has 112 cm. Guessing the system based on routing produced wrong garments for multi-facility accounts and was non-deterministic. As of 2.4.0 the API refuses to guess when routing is ambiguous.

**Affected clients:** printf-js, printf-py, printf-java, printf-rb, printf-go — update to the 2.4.x release of your library to get the new fields in types and response models.

### Upgrade path

Add `size_system` to every order and every line. See the migration note for a before/after payload and the full remediation steps.

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
