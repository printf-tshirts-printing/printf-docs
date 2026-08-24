---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform-docs
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

This guide gets you to a working order in five minutes. It reflects **order-api 2.4.0**. If you are on an earlier version, the `size_system` and `fit` fields do not exist yet — upgrade before following this guide.

## Prerequisites

- An API key with `orders:write` scope.
- Your `accountId` (starts with `acct_`).
- At least one `designId` and one `facilityId`.

## Step 1 — Install the SDK

Pick your language:

```bash
# JavaScript / TypeScript
npm install printf-js

# Python
pip install printf-py

# Java (Maven)
# add com.printf:printf-java:2.4.x to your pom.xml

# Go
go get github.com/printf-dev/printf-go

# Ruby
gem install printf-rb
```

## Step 2 — Place your first order

The minimum viable request requires `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, `quantity`, `size`, `size_system`, and `fit`.

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

## Step 3 — Read the response

A successful response is `201 Created`. Each line includes a `resolved_size` object so you can confirm the system interpreted your size label the way you intended:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The response also sets `X-Printf-Size-System` to the system that was applied. Log this header — it is the fastest way to diagnose a sizing mismatch without re-fetching the order.

## Step 4 — Handle errors

| Code | HTTP status | What it means |
|---|---|---|
| `size_system_ambiguous` | 400 | Your account routes to multiple facilities and you did not declare a `size_system`. Add it to the order or to each line. |
| `size_system_implicit` | 200 (warning) | Your account is single-facility and the system was inferred from the facility default. Add `size_system` explicitly — this becomes an error in 2.6. |

Always check the `warnings` array in 200 responses before assuming an order is clean.

## Next steps

- **Sizing and fit** (`docs/guides/sizing.md`) — full reference for `size_system`, `fit`, and `resolved_size`.
- **Bulk orders and templates** (`docs/guides/bulk-orders.md`) — submit multiple orders at once and manage saved templates.

