---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released 2026-08-24**

### Breaking change

`size` on order lines is no longer interpreted without an explicit size system. Requests that omit `size_system` on both the line and the order are rejected with `400 size_system_ambiguous` for accounts that can route to more than one facility.

### New fields

| Location | Field | Type | Description |
|---|---|---|---|
| Request — order | `size_system` | `US` \| `EU` \| `JP` | Default size system for all lines |
| Request — line | `size_system` | `US` \| `EU` \| `JP` | Per-line override |
| Request — line | `fit` | string | Garment fit (e.g. `unisex`, `womens`) |
| Response — line | `resolved_size` | object | `{ label, system, fit, chest_cm }` |
| Response — header | `X-Printf-Size-System` | string | The size system applied to the order |

### Size ladder differences

Ladders differ materially between systems. A JP `XL` resolves to 97 cm chest; a US `XL` resolves to 112 cm. Wrong system = wrong garment at the door.

### Fallback chain (bare labels)

Line → order → account → facility default. Accounts routable to more than one facility skip the facility default and return `400 size_system_ambiguous`.

### Warning → error timeline

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning now. This becomes a hard error in **2.6**.

### Affected SDKs

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — update to the 2.4.x releases of each library for typed `size_system` and `fit` fields and `resolved_size` response models.

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
