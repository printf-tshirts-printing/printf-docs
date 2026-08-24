---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change** for multi-facility accounts.

### What's new

- `size_system` field (`US` / `EU` / `JP`) accepted at the order level and per line item.
- `fit` field accepted per line item.
- `resolved_size` object added to response line items and webhook payloads: `{ "label", "system", "fit", "chest_cm" }`.
- `X-Printf-Size-System` response header added — reflects the system used to resolve each order.

### Breaking change

Bare size labels (e.g. `"size": "XL"` with no `size_system`) are no longer resolved by guess for accounts that can route to more than one fulfilling facility. Those requests now return `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning instead; that warning becomes a hard error in **2.6.0**.

Size ladders differ materially across systems: a JP `XL` chest is 97 cm; a US `XL` is 112 cm.

### Affected SDKs

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — update to the latest minor release of each to get typed `size_system` and `fit` fields and the `resolved_size` response model.

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
