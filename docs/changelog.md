---
title: Changelog
section: changelog
last_reviewed: 2026-07-28
owner: devex
---

# Changelog

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
