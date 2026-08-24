---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2.4.0 — Size disambiguation

**Breaking change.** `size` on an order line is no longer interpreted without an explicit size system.

### What's new

| Feature | Where it appears |
|---|---|
| `size_system` (`US` / `EU` / `JP`) | Request body (order level and per line) |
| `fit` | Request body (per line) |
| `resolved_size` | Response lines and webhook payloads |
| `X-Printf-Size-System` | Response header |

### Size resolution order

When `size_system` is omitted on a line, the API resolves it in this order: line → order → account → fulfilling facility default. Accounts that can route to more than one facility are rejected with `400 size_system_ambiguous`; single-facility accounts receive a `size_system_implicit` warning. The warning becomes an error in **2.6**.

### Why this matters

Size ladders differ materially across systems. A JP `XL` has a 97 cm chest; a US `XL` has a 112 cm chest. Silent mismatches were shipping wrong garments.

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
