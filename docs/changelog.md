---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

**Order API 2.4.0 — Size disambiguation (breaking change)**

Sizes are no longer interpreted without a system. You must now supply `size_system` (`US`, `EU`, or `JP`) on every order or line.

**New fields**

| Field | Location | Required |
|---|---|---|
| `size_system` | Order body | Recommended (see below) |
| `size_system` | Line object | Recommended (see below) |
| `fit` | Line object | Optional |
| `resolved_size` | Response line / webhook payload | — |
| `X-Printf-Size-System` | Response header | — |

**Resolution cascade** (line → order → account → facility default) only works when your account routes to exactly one facility. Multi-facility accounts that omit `size_system` receive `400 size_system_ambiguous`. Single-facility accounts receive a `size_system_implicit` warning, which becomes a hard error in 2.6.0.

Size ladders differ materially across systems — a JP `XL` is 97 cm chest, a US `XL` is 112 cm. Ambiguity is not silent: it is now a controlled error.

Affected SDKs: printf-js, printf-py, printf-java, printf-rb, printf-go.

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
