---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size system (2026-08-20)

Size labels are no longer ambiguous. `POST /v2/orders` now accepts `size_system` (`US` / `EU` / `JP`) at the order level and per line, plus `fit` per line. Response lines and webhook payloads gain `resolved_size`. A new `X-Printf-Size-System` response header echoes the resolved system.

**Breaking change for multi-facility accounts:** requests that omit `size_system` and can route to more than one facility are rejected with `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning today; that warning becomes an error in 2.6.

Ladders differ materially: JP `XL` = 97 cm chest, US `XL` = 112 cm chest.

All five SDKs (`printf-js`, `printf-py`, `printf-go`, `printf-java`, `printf-rb`) have `supportsSizeSystem=false` and will require updates before size-system fields are usable through them.

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
