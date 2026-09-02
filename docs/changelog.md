---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

## 2026-09-02 — Orders API 2.4.0

**Size disambiguation. Breaking change for accounts routing to more than one facility.**

- **Breaking:** `POST /v2/orders` no longer resolves a bare size label by guessing the fulfilling facility's default. Accounts that can route to more than one facility now receive `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning — this becomes an error in 2.6.
- `size_system` (`US`, `EU`, `JP`) is accepted on the order and on each line; the line value takes precedence.
- `fit` is accepted per line.
- Responses and webhook payloads carry `resolved_size` per line (`label`, `system`, `fit`, `chest_cm`).
- Responses carry a new `X-Printf-Size-System` header reflecting the system applied.
- JP `XL` is 97 cm; US `XL` is 112 cm. Omitting `size_system` on a saved template or an existing integration will produce the wrong garment for cross-system accounts.

[Migration note and diff](/guides/sizing#migrating-to-explicit-size-system).

## 2026-09-02 — Orders API 2.4.0

**Size disambiguation. Breaking change for multi-facility accounts.**

- **Breaking:** Accounts routing to more than one facility now receive `400 size_system_ambiguous` instead of a facility-default guess when no `size_system` is supplied.
- `POST /v2/orders` accepts `size_system` (`US`, `EU`, `JP`) at the order level and per line; `fit` is accepted per line.
- Responses and webhook payloads carry `resolved_size` per line. A new `X-Printf-Size-System` response header echoes the resolved system.
- Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning beginning in 2.4.0. This becomes an error in 2.6.
- Size ladders differ materially across systems: JP XL is 97 cm chest; US XL is 112 cm.

Saved order templates carry no explicit `size_system` and will hit `size_system_ambiguous` if the account routes to more than one facility. [What to change](/guides/sizing#migrating-saved-templates).

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
