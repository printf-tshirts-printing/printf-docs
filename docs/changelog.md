---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation (breaking)

**Released:** 2026-08-24

### What changed

Size labels now require an explicit size system. `size_system` (`US` / `EU` / `JP`) is accepted on the order and per line item. Each response line now includes `resolved_size` with the normalised label, system, fit, and chest measurement in centimetres. The `X-Printf-Size-System` response header echoes the system used for the order.

### Breaking change

Bare size labels (no `size_system` anywhere in the request) are **no longer resolved by guess**. Accounts that can route to more than one facility receive `400 size_system_ambiguous`. Single-facility accounts still resolve implicitly but receive a `size_system_implicit` warning — this becomes a hard error in 2.6.

### Why this matters

JP XL (97 cm chest) and US XL (112 cm chest) differ by a full size step. Silent misresolution was producing wrong-size fulfilment for multi-facility accounts.

### New fields at a glance

| Location | Field | Type | Notes |
|---|---|---|---|
| Order request | `size_system` | `"US"\|"EU"\|"JP"` | Default for all lines |
| Line request | `size_system` | `"US"\|"EU"\|"JP"` | Overrides order-level |
| Line request | `fit` | `"unisex"\|"mens"\|"womens"` | Optional |
| Line response | `resolved_size` | object | `label`, `system`, `fit`, `chest_cm` |
| Response header | `X-Printf-Size-System` | string | Echoes resolved system |

### Affected SDKs

printf-js, printf-py, printf-java, printf-go, printf-rb — all updated to 2.4.0.

### Action required

See the [migration note](#migration) for before/after payloads and the deprecation timeline.

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
