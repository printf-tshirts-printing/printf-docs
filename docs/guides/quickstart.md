---
title: Quickstart
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Quickstart

This guide gets you from zero to a confirmed order in five minutes.

## Prerequisites

- An API key for your account (`acct_...`).
- A design ID (`dsn_...`) uploaded via the dashboard or the Designs API.
- A garment SKU — see the [Catalog](../api/catalog.md) reference for available values.

## 1 — Create your first order

Send a `POST /v2/orders` request. Every request requires `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, `quantity`, `size`, and `size_system`.

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "size": "XL",
      "size_system": "US",
      "fit": "unisex"
    }
  ]
}
```

> **`size_system` is required as of 2.4.0.** Omitting it on a multi-facility account returns `400 size_system_ambiguous`. On a single-facility account it returns a `size_system_implicit` warning today and will become a `400` error in 2.6.0. Always set it explicitly.

## 2 — Read the response

A successful `201` response includes a `resolved_size` object on each line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Check `resolved_size.system` and `resolved_size.chest_cm` to confirm the garment you ordered. Size ladders differ materially: a JP `XL` is 97 cm, a US `XL` is 112 cm.

The response also includes the `X-Printf-Size-System` header, reflecting the system applied to the whole request.

## 3 — Next steps

- [Sizing and fit](sizing.md) — full documentation of size systems, the resolution ladder, `fit`, and `resolved_size`.
- [Bulk orders and templates](bulk-orders.md) — sending multiple lines, and updating saved templates to include `size_system`.
- [Webhooks](../api/webhooks.md) — `resolved_size` also appears in `order.created` and `order.updated` webhook payloads.

