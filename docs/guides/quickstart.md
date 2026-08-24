---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

Send your first order in under five minutes.

> **If you are upgrading from a pre-2.4.0 integration**, read the [sizing and fit guide](./sizing.md) before updating your code. `size` without `size_system` is now a warning or error depending on your account configuration.

## Prerequisites

- A Printf account (`accountId` from the dashboard)
- An API key (Settings → API keys)
- A design uploaded and approved (`designId`)

## Step 1 — Install the client library

Pick your language:

```bash
# JavaScript / TypeScript
npm install printf-js

# Python
pip install printf-py

# Java (Maven)
# add com.printf:printf-java:2.4.0 to pom.xml

# Go
go get github.com/printf-dev/printf-go@v2.4.0

# Ruby
gem install printf-rb
```

Use the **2.4.x** release of your library to get `size_system`, `fit`, and `resolved_size` support.

## Step 2 — Send an order

```json
POST /v2/orders
Authorization: Bearer <your-api-key>
Content-Type: application/json

{
  "accountId": "<your-accountId>",
  "size_system": "US",
  "facilityId": "<your-facilityId>",
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

`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every request.

`size_system` is required as of 2.4.0. Set it at the order level as a default, and override per line when you are mixing systems.

## Step 3 — Check the response

A successful response includes `resolved_size` on every line:

```json
{
  "orderId": "ord_abc123",
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "resolved_size": {
        "label": "XL",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 112
      }
    }
  ]
}
```

Also check the `X-Printf-Size-System` response header. It reflects the size system applied to the order. If it does not match your intent, do not proceed — cancel the order and resend with an explicit `size_system`.

## Error codes you will see during migration

| Code | HTTP status | What to do |
|---|---|---|
| `size_system_ambiguous` | 400 | Add `size_system` to the order or the affected line |
| `size_system_implicit` | warning only (400 in 2.6) | Add `size_system` before upgrading to 2.6 |

## Next steps

- [Sizing and fit](./sizing.md) — full size-system reference and ladder tables
- [Bulk orders and templates](./bulk-orders.md) — multi-line orders and saved template migration

