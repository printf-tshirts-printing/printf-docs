---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## order-api 2.4.0 — Size disambiguation

**Released: 2026-08-24**

### Breaking change

`size` on order lines is no longer interpreted without an explicit size system. Requests that omit `size_system` on both the line and the order are rejected with `400 size_system_ambiguous` if the account can route to more than one fulfilling facility.

### What's new

| Field / header | Where | Description |
|---|---|---|
| `size_system` | Request body (order + line) | `US`, `EU`, or `JP`. Line value overrides order value. |
| `fit` | Request line | `unisex`, `mens`, `womens` |
| `resolved_size` | Response line + webhook payload | `{ "label", "system", "fit", "chest_cm" }` |
| `X-Printf-Size-System` | Response header | The system used to resolve every line in the request. |

### Warning → error timeline

Accounts that omit `size_system` but route to exactly one facility today receive a `size_system_implicit` warning. This warning becomes a hard error in **2.6**.

### Size ladder note

Ladders differ materially across systems: a JP `XL` resolves to 97 cm chest; a US `XL` resolves to 112 cm. Double-check saved order templates if you have historically shipped to Japan or EU.

### Affected SDKs

printf-js · printf-py · printf-java · printf-rb · printf-go

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
