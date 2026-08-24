---
title: Quickstart
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Quickstart

This guide gets you from zero to a confirmed order in under ten minutes.

## Prerequisites

- An active Printf account and API key.
- Your `accountId`, available on the Accounts page in the dashboard.
- A design uploaded via the dashboard or `POST /v2/designs`. You need its `designId`.

## 1. Place your first order

Send a `POST /v2/orders` request. `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every request. `size_system` is required on any account that routes to more than one fulfillment facility (see [Sizing and fit](./sizing.md)).

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

## 2. Read the response

A `201 Created` response contains the order object. Each line now includes `resolved_size`, confirming what the fulfillment system will use:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header shows the system applied at the order level. Check it against your expectation before proceeding.

## 3. Handle errors

| Code | What it means | Fix |
|---|---|---|
| `size_system_ambiguous` | Your account routes to multiple facilities and no `size_system` was found | Add `size_system` to the order or each line |
| `size_system_implicit` | Your account is single-facility and relied on the facility default | Add `size_system` — required before 2.6 |

## 4. Next steps

- [Sizing and fit](./sizing.md) — full ladder tables, fit options, and fallback rules.
- [Bulk orders and templates](./bulk-orders.md) — place multiple orders in one call and audit saved templates.

