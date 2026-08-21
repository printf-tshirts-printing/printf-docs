---
title: Webhooks
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks: []
---

# Webhooks

Printf sends webhook events to your registered endpoint as orders move through fulfillment. This page covers the event schema as of **order-api 2.4.0**.

## New in 2.4.0: `resolved_size` on line items

Every `order.accepted`, `order.fulfilled`, and `order.shipped` event now includes `resolved_size` on each line item:

```json
{
  "event": "order.accepted",
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

`resolved_size` is always present on lines in webhook payloads from 2.4.0. If your webhook consumer stores or forwards line data, update your schema to accept the new object. It is **additive** — existing fields are unchanged.

### `resolved_size` fields

| Field | Type | Description |
|-------|------|-------------|
| `label` | string | The size label as submitted |
| `system` | string | `US`, `EU`, or `JP` — the system that was applied |
| `fit` | string | The fit applied to this line |
| `chest_cm` | integer | Chest measurement in cm |

## Signature verification

All webhook deliveries include an `X-Printf-Signature` header. Verify it before processing:

```
X-Printf-Signature: sha256=<hmac>
```

Compute `HMAC-SHA256(secret, raw_request_body)` and compare. Reject requests where the signatures do not match.

## Retry behavior

If your endpoint returns a non-`2xx` status, Printf retries with exponential backoff: 10 s, 30 s, 5 min, 30 min, 2 h. After five failures the event is marked undeliverable and you can replay it from the [dashboard](https://app.printf.dev/settings/webhooks).

## Event types

| Event | Fired when |
|-------|------------|
| `order.accepted` | Order passes validation and enters the fulfillment queue |
| `order.fulfilled` | Garments are cut and packed |
| `order.shipped` | Carrier scan recorded |
| `order.cancelled` | Order cancelled before fulfillment |

