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

This guide covers high-volume order submission and the saved-template workflow. If you use templates created before **order-api 2.4.0**, read the [sizing and fit guide](sizing.md) and the note on templates below — legacy templates will fail or warn on use if they omit `size_system`.

## Submitting a bulk order

Bulk orders use the same `POST /v2/orders` endpoint as single orders. There is no separate bulk endpoint. Add as many objects to `lines` as you need; each line is fulfilled independently.

Every request must include `accountId`, `destination`, and at least one line. Each line must include `designId`, `garmentSku`, and `quantity`.

As of 2.4.0, you must also provide `size_system` — either at the order level as a default or on every line individually. See the [sizing and fit guide](sizing.md) for the full resolution chain.

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
      "garmentSku": "tee-classic-white"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "EU",
      "fit": "unisex",
      "quantity": 75,
      "garmentSku": "tee-classic-navy"
    }
  ]
}
```

Note the third line uses `"size_system": "EU"` — line-level `size_system` overrides the order-level default of `US`.

## Interpreting bulk responses

Each line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For large runs, iterate the response lines and assert that every `resolved_size.system` and `chest_cm` value matches your expectations before treating the order as confirmed. A mis-sized bulk run is expensive to reverse.

## Saved order templates

Templates let you store a partial or complete order body under a name and reuse it. They are not validated when saved — only when used to create an order.

**Templates created before 2.4.0 omit `size_system` and will behave as follows when used:**

| Account routing | Result |
|---|---|
| Routes to exactly one facility | Order created with `size_system_implicit` warning. Becomes an error in 2.6. |
| Routes to more than one facility | Order rejected with `400 size_system_ambiguous`. |

**You must update every saved template before 2.6.** Add `size_system` at the order level, and add `size_system` and `fit` to each line as appropriate. There is no API endpoint that lists templates — retrieve them from your own configuration store or the Printf dashboard.

## Rate limits and retries

Bulk requests with many lines are still a single API call and count as one request against your rate limit. If a request fails, the entire order fails — there is no partial-success response. On retry, submit the complete original payload with the corrected `size_system` fields.

## Webhooks

Webhook payloads for order events now include `resolved_size` on every line, matching the structure in the synchronous response. Update any webhook consumers that parse line items to handle the new field.

