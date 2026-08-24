---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

`size` now requires an explicit size system. This is a breaking change for any integration that omits `size_system`.

### What's new

| Feature | Details |
|---|---|
| `size_system` field | Accepted on the order root and per line item. Values: `US`, `EU`, `JP`. Line value overrides order value. |
| `fit` field | Per-line field. Values: `unisex`, `mens`, `womens`. |
| `resolved_size` on responses | Every line in the response and in webhook payloads now includes a `resolved_size` object: `{ "label", "system", "fit", "chest_cm" }`. |
| `X-Printf-Size-System` header | Response header echoing the resolved system for every order. |

### Breaking change

Bare size labels (`"size": "XL"` with no `size_system` anywhere) are no longer resolved by guess.

- **Accounts that can route to more than one facility** → `400 size_system_ambiguous`. Requests are rejected immediately.
- **Single-facility accounts** → request succeeds but returns a `size_system_implicit` warning. This warning becomes a hard error in **2.6**.

Size ladders differ materially across systems: JP `XL` is 97 cm chest, US `XL` is 112 cm. Silent fallback was a mis-fulfillment risk.

### Affected client libraries

`printf-js` · `printf-py` · `printf-java` · `printf-go` · `printf-rb`

Updated SDK releases ship alongside this API version. Pin to the new major if you are on a fixed version.

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
