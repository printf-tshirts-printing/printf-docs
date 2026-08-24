---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released:** 2026-08-24

### Breaking change

`size` fields on order lines are no longer interpreted without an explicit size system. Requests that omit `size_system` on both the order and the affected line are rejected with `400 size_system_ambiguous` if the account can route to more than one facility. Single-facility accounts receive a `size_system_implicit` warning today; this becomes a hard error in 2.6.

### What's new

| Field / header | Where | Notes |
|---|---|---|
| `size_system` | Order root + each line | Accepted values: `US`, `EU`, `JP` |
| `fit` | Each line | e.g. `unisex`, `womens`, `mens` |
| `resolved_size` | Response lines + webhook payloads | `{ "label", "system", "fit", "chest_cm" }` |
| `X-Printf-Size-System` | Response header | The system applied to the order |

### Why it matters

Size ladders differ materially across systems. A JP `XL` chest is 97 cm; a US `XL` is 112 cm. Silent mis-routing was producing wrong-size fulfillment with no error signal.

### Affected client libraries

`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb` — update to the 2.4.x release of your library before deploying integrations against this API version.

### Keynote accounts

StackFest, Cloud Native Rodeo, ObservaCon, KubeSummit, and ShipItConf are on this release. If you are on a managed plan, contact support before 2.6 ships to avoid order rejections.

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
