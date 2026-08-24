---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers submitting large orders and reusing saved order templates. As of **order-api 2.4.0**, both must include `size_system` — templates that omit it will fail on first use for multi-facility accounts.

## Bulk order structure

There is no separate bulk endpoint. A single `POST /v2/orders` call accepts multiple lines. Set `size_system` at the order level to apply one system to all lines, or override per line.

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
      "fit": "unisex",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "fit": "unisex",
      "quantity": 150,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

In this example the first two lines inherit `US` from the order. The third line overrides to `JP` — its `resolved_size.chest_cm` in the response will be 97, not 112.

## Reading resolved sizes in bulk responses

The response includes `resolved_size` on every line:

```json
{
  "label": "XL",
  "system": "JP",
  "fit": "unisex",
  "chest_cm": 97
}
```

For bulk orders, iterate every line and compare `resolved_size.chest_cm` against your design specification before confirming dispatch. A mismatch here — not at placement time — is the last point at which fulfilment can be corrected.

## Saved order templates

> **Action required before 2.6.0.** Templates that omit `size_system` will fail immediately with `400 size_system_ambiguous` for multi-facility accounts, and will emit `size_system_implicit` warnings for single-facility accounts. That warning becomes an error in **2.6.0**.

Templates are not returned by any API call. To audit them:

1. Log in to the Printf dashboard.
2. Navigate to **Account → Order templates**.
3. Open each template and add `size_system` at the order level, or per line if the template mixes size ladders.
4. Save and re-validate by placing a test order before updating your production account.

Templates used by event management workflows — where addresses and quantities vary but garments are fixed — are especially likely to omit `size_system` because the field did not exist when they were created.

## Idempotency for bulk submissions

Set the `Idempotency-Key` header on every bulk submission. Network retries on large payloads are common; without an idempotency key a retry creates a duplicate order.

```
Idempotency-Key: bulk-stackfest-2026-082401
```

Keys are scoped to your `accountId` and expire after 24 hours.

## Rate limits

Bulk calls count as one request regardless of line count, but each line is evaluated against the line-rate limit of 500 lines per call. Exceeding this returns `400 too_many_lines`.

## Errors relevant to bulk orders

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | At least one line (or the order) has no `size_system` and the account routes to multiple facilities. Add `size_system` to the order or the affected lines. |
| `size_system_implicit` | — | Warning: one or more lines resolved `size_system` from the facility default. Becomes an error in **2.6.0**. |
| `too_many_lines` | 400 | Payload exceeds 500 lines. Split into multiple requests. |

