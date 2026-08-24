---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-rb
  - printf-go
---

# Bulk orders and templates

**Updated for order-api 2.4.0.** Saved order templates are affected by the size-disambiguation change. Read the [Templates and size system](#templates-and-size-system) section before submitting bulk orders.

## Sending a bulk order

A bulk order is a single `POST /v2/orders` call whose `lines` array contains multiple entries. There is no separate bulk endpoint. The same required fields apply to every line regardless of quantity.

Required fields on every request:

| Field | Level | Notes |
|---|---|---|
| `accountId` | Order | |
| `destination` | Order | Full address object |
| `designId` | Line | |
| `garmentSku` | Line | |
| `quantity` | Line | |

Recommended from 2.4.0 onwards (required for multi-facility accounts):

| Field | Level | Notes |
|---|---|---|
| `size_system` | Order and/or line | `US`, `EU`, or `JP` |
| `fit` | Line | `unisex`, `mens`, or `womens` |

**Example — bulk order with mixed lines**

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
      "size": "S",
      "size_system": "US",
      "fit": "womens",
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
```

Each line in the response includes a `resolved_size` object confirming the label, system, fit, and chest measurement that was used to fulfil that line. Verify these before sending order confirmations to recipients.

## Templates and size system

Saved order templates are **not visible in the API** — they live in the dashboard and are applied at submission time. If your templates include bare size labels (e.g. `"size": "XL"` without a `size_system`), they are now subject to the 2.4.0 disambiguation rules:

- **Multi-facility accounts**: templates with bare size labels will be rejected with `400 size_system_ambiguous` immediately.
- **Single-facility accounts**: templates with bare size labels will produce a `size_system_implicit` warning per request. This becomes a hard error in **2.6**.

**What to do:**

1. Open the dashboard and review every saved template.
2. Add `size_system` at the order level of each template (e.g. `"size_system": "US"`).
3. Optionally add `size_system` and `fit` per line for precision.
4. Re-save each template.

You cannot patch templates via the API. Changes must be made in the dashboard.

## Handling `resolved_size` in bulk responses

For large orders, iterate `lines[].resolved_size` to build a per-line fulfilment manifest before dispatch. The `chest_cm` field is the single authoritative measurement; size labels are human-readable shorthand only.

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

## Webhook payloads

Bulk order webhooks include `resolved_size` on every line in the payload, identical to the synchronous response. No webhook schema changes are required on your end, but if you parse size out of the bare `size` field today you should switch to `resolved_size.label` (and check `resolved_size.system`) so your fulfilment logic is system-aware.

## Error reference

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Bare size label, multi-facility account | Add `size_system` to the order or affected lines |
| `size_system_implicit` | — (warning) | Bare size label, single-facility account | Add `size_system` before 2.6 |

