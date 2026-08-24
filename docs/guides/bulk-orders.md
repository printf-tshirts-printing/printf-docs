---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - POST /v2/orders/bulk
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

Bulk orders let you submit many lines — or many orders — in a single request. Saved order templates let you reuse a base payload without repeating common fields.

> **2.4.0 action required for templates.** Saved templates do not inherit the new `size_system` field automatically. If your templates omit `size_system`, orders submitted from them will hit the fallback resolution chain described in the [Sizing and fit](sizing.md) guide. Multi-facility accounts will receive `400 size_system_ambiguous`. Update your saved templates before upgrading to 2.4.0.

## Bulk order structure

A bulk order is an array of standard order objects. Each object must include all required fields: `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`.

As of 2.4.0, you should also include `size_system` at the order level or on each line.

```json
POST /v2/orders/bulk
[
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
        "size": "M",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 100,
        "garmentSku": "tee-classic-black"
      },
      {
        "designId": "dsn_7fa91c",
        "size": "XL",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 150,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
]
```

## Saved order templates

Saved templates store a base order payload that you can reference by template ID at submission time. They are not visible in the catalog API — you manage them through the dashboard or the `/v2/order-templates` endpoints.

### Updating templates for 2.4.0

Templates created before 2.4.0 have no `size_system` field. When a template-based order is submitted:

- If your account has a single fulfilling facility, the order resolves via facility default and emits a `size_system_implicit` warning.
- If your account can route to more than one facility, the order is rejected with `400 size_system_ambiguous`.

To update a saved template, retrieve it, add `size_system` at the order level (and optionally per line), then save it back. The template body follows the same shape as a direct POST:

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "destination": { ... },
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

## Partial failure handling

Bulk submissions are validated line by line. If one order in the batch triggers `size_system_ambiguous`, that order is rejected and the rest of the batch proceeds. The response body lists each order's status and any error codes. Check `size_system_ambiguous` per order, not only at the top level.

## Responses and webhooks

Every fulfilled line now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Webhook payloads for order confirmation events carry the same `resolved_size` structure on each line. If you log or forward webhook payloads to a warehouse or fulfilment system, make sure those consumers can handle the new field without rejecting the event.

## Error codes reference

| Code | Status | Scope | Cause |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Per order | Multi-facility account, no size system resolved |
| `size_system_implicit` | warning | Per order | Single-facility account, resolved from facility default — becomes error in 2.6 |

