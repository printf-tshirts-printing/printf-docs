---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: ["POST /v2/orders"]
covers_sdks: ["printf-js","printf-py","printf-java","printf-go","printf-rb"]
---

# Bulk orders and templates

## ⚠️ Action required — `size_system` (2.4.0)

Saved order templates that omit `size_system` will produce `size_system_implicit` warnings from order-api **2.4.0** and will be **rejected as errors in 2.6**. Templates are not updated automatically and are invisible from the list-orders API, so check each template directly.

Add `size_system` at the order level in every template:

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "facilityId": "fac-atx",
  "destination": { ... },
  "lines": [ ... ]
}
```

See [Sizing and fit](sizing.md) for the full fallback chain and per-line overrides.

---

## Creating a bulk order

Send multiple lines in a single `POST /v2/orders` request. All lines share the order-level `size_system` unless overridden per line.

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
      "garmentSku": "tee-classic-black",
      "size": "S",
      "fit": "unisex",
      "quantity": 100
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "M",
      "fit": "unisex",
      "quantity": 200
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "L",
      "fit": "unisex",
      "quantity": 250
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

Each line in the response includes `resolved_size`:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

## Using saved templates

Templates are referenced by ID when creating an order. Because templates pre-populate fields that are not visible in the API response for list-orders, audit them directly in your integration to confirm `size_system` is set.

## Error codes

| Code | HTTP | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` provided; request rejected |
| `size_system_implicit` | — | Warning: single-facility account, no `size_system`; facility default used; **becomes error in 2.6** |

