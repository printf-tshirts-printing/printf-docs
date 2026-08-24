---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Quickstart

This guide gets you from zero to a confirmed order in under ten minutes. It uses order-api **2.4.0**. If you are on an older client library, upgrade first — size fields added in 2.4 are required.

## Before you start

You need:

- An API key (Settings → API keys in the dashboard).
- Your `accountId` (shown on the account overview page).
- Your `facilityId` if you route to a specific facility.
- A `designId` for at least one approved design.

## Step 1 — Install the SDK

```bash
# JavaScript / TypeScript
npm install printf-js@^2.4.0

# Python
pip install printf-py>=2.4.0

# Java (Maven)
# <dependency><groupId>dev.printf</groupId><artifactId>printf-java</artifactId><version>[2.4.0,)</version></dependency>

# Go
go get dev.printf/printf-go@v2.4.0

# Ruby
gem install printf-rb -v '~> 2.4'
```

## Step 2 — Place your first order

The minimal working request. All five required fields are present: `accountId`, `destination`, `designId`, `garmentSku`, and `quantity`. `size_system` is required from 2.4.0 — omit it and single-facility accounts get a warning, multi-facility accounts get a `400`.

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "facilityId": "fac-atx",
  "destination": {
    "name": "StackFest Ops",
    "line1": "410 Congress Ave",
    "city": "Austin",
    "region": "TX",
    "postalCode": "78701",
    "countryCode": "US"
  },
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### What comes back

The response includes a `resolved_size` object on every line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response header `X-Printf-Size-System` confirms the effective size system applied to the order. Log it for auditing.

## Step 3 — Handle errors

| Code | What it means | Fix |
|---|---|---|
| `400 size_system_ambiguous` | Your account routes to multiple facilities and `size_system` is missing | Add `size_system` to the order or each line |
| `size_system_implicit` (warning) | `size_system` resolved via single-facility default | Add `size_system` explicitly before 2.6 |

## Next steps

- [Sizing and fit](sizing.md) — full ladder tables, `fit` values, and resolution rules.
- [Bulk orders and templates](bulk-orders.md) — batch payloads and how to update saved templates.

