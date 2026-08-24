---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

This guide covers sending multi-line orders and using saved order templates. Both are affected by the size-system changes in **2.4.0**.

## Multi-line orders

Each line in `lines[]` is fulfilled independently. As of 2.4.0, set `size_system` and `fit` per line to be explicit — mixing EU and US lines in a single order is valid, and line-level `size_system` overrides the order-level value.

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
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "EU",
      "fit": "womens",
      "quantity": 150,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Both lines return `resolved_size` independently, with the system and `chest_cm` that actually drove fulfillment. Check both before committing large runs.

## Saved order templates

> **Action required if you use saved templates.**

Saved order templates store line definitions including `size` labels. Templates created before 2.4.0 do not contain `size_system`. When a template is submitted:

- If your account is single-facility: the order is accepted with a `size_system_implicit` warning.
- If your account routes to more than one facility: the order is **rejected** with `400 size_system_ambiguous`.

**Templates are not visible via the API.** You must edit them in the dashboard or re-create them via `POST /v2/orders` and save as a new template with explicit `size_system` and `fit` on every line.

This is the most common source of silent breakage after upgrading. Check all templates before 2.6 ships — `size_system_implicit` becomes a hard error at that version, affecting single-facility accounts too.

## Webhook payloads for bulk orders

Each line in the webhook payload now includes `resolved_size`. Use this to reconcile fulfillment against your purchase order:

```json
{
  "event": "order.fulfilled",
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 100,
      "resolved_size": {
        "label": "L",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 104
      }
    }
  ]
}
```

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no resolvable size system |
| `size_system_implicit` | — (warning) | Single-facility account, size system inferred from facility default. Becomes a 400 in 2.6 |

