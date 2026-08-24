---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

Size fields are now explicit. `size_system` (`US` / `EU` / `JP`) is accepted on the order object and per line item. Each response line now includes a `resolved_size` object, and the `X-Printf-Size-System` header echoes the system used for the order.

Bare size labels still resolve through a fallback chain (line → order → account → facility default). Accounts that can route to more than one facility can no longer rely on implicit facility-default resolution and will receive `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning today; that warning becomes a hard error in **2.6**.

Ladder differences are material: JP `XL` = 97 cm chest, US `XL` = 112 cm chest.

**New fields:** `size_system` (order + line), `fit` (line), `resolved_size` (response line + webhook payload).
**New header:** `X-Printf-Size-System`.
**New error:** `400 size_system_ambiguous`.
**New warning:** `size_system_implicit` (error in 2.6).
**Affected SDKs:** printf-js, printf-py, printf-java, printf-go, printf-rb.

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
