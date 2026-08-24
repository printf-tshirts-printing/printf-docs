---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers high-volume order submission and the use of saved order templates. If you are submitting large conference orders — such as those for StackFest, Cloud Native Rodeo, or KubeSummit — read the size-system section below before submitting.

## Submitting a bulk order

Bulk orders use the same `POST /v2/orders` endpoint as single orders. Each line item is an independent entry in the `lines` array.

**Required fields on every request:** `accountId`, `destination`, and at least one line containing `designId`, `garmentSku`, and `quantity`.

As of **2.4**, you must also supply `size_system` at the order level or per line. See [Sizing and fit](./sizing.md) for the full resolution chain.

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
      "fit": "unisex",
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## Verifying resolved sizes

Every line in the response now includes `resolved_size`:

```json
{ "label": "M", "system": "US", "fit": "unisex", "chest_cm": 104 }
```

For bulk orders, iterate `lines[].resolved_size.chest_cm` before confirming production. A system mismatch on 500 shirts is not a cheap mistake.

## Saved order templates

> **Action required before 2.6.** Templates are not visible through the API. You must audit them in the dashboard.

If your bulk workflow uses saved templates, each template needs `size_system` added at the order level or per line:

| Template state | Behaviour in 2.4 | Behaviour in 2.6 |
|---|---|---|
| Has explicit `size_system` | ✅ Resolves cleanly | ✅ Resolves cleanly |
| Missing `size_system`, single-facility account | ⚠️ `size_system_implicit` warning | ❌ `400` error |
| Missing `size_system`, multi-facility account | ❌ `400 size_system_ambiguous` | ❌ `400 size_system_ambiguous` |

To update a template: open the dashboard, find the template under **Order templates**, and add `size_system` to the order body and to each line that does not already carry it.

## Error handling for bulk submissions

Bulk requests are atomic — a single invalid line rejects the entire order. Check for these codes:

| Code | HTTP status | Likely cause in a bulk order |
|---|---|---|
| `size_system_ambiguous` | 400 | At least one line (or the order) is missing `size_system` and the account routes to multiple facilities. |
| `size_system_implicit` | — (warning in 2.4) | `size_system` resolved via account or facility default. Fix before 2.6. |

## Rate limits

Rate limits apply per account, not per line. A single request with 500 lines counts as one request against your quota.
