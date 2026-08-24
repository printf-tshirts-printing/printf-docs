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

This guide covers creating large orders and using saved order templates. **If you use templates, read the sizing section below before upgrading to 2.4.0** — templates are not visible from the API and you may have saved bare size labels in them that will now trigger errors.

## Creating a bulk order

Bulk orders follow the same endpoint as single-item orders. The `lines` array accepts up to 500 entries per request.

As of **2.4.0**, every line must resolve to an unambiguous size system. Set `size_system` at the order root to apply one system to all lines, or set it per line for mixed-market runs.

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
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 180,
      "garmentSku": "tee-classic-black"
    },
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

Verify the `resolved_size` object on each line of the response before treating an order as confirmed. It tells you the exact chest measurement the fulfillment system recorded:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

## Saved order templates

Templates are stored server-side and are **not** returned by any list endpoint. If you created templates before 2.4.0, they likely contain bare size labels with no `size_system`.

### What happens when you submit a template-based order in 2.4.0

| Account type | Outcome |
|---|---|
| Single-facility account | Order is accepted. Each affected line carries a `size_system_implicit` warning. The inferred system is logged in `X-Printf-Size-System`. **This becomes a `400` error in 2.6.** |
| Multi-facility account | Order is **rejected** with `400 size_system_ambiguous`. No fulfillment occurs. |

### How to update your templates

There is no bulk-template API. You must update each template through the Printf dashboard or re-submit the template definition via `POST /v2/order-templates` with the `size_system` and `fit` fields added.

For every line in every template:

1. Add `"size_system"` matching the market the garment is cut for.
2. Add `"fit"` if the template targets a non-unisex cut. If omitted, `unisex` is assumed.

### Checking your templates proactively

Submit a test order (quantity 1, a known address) using each template before your next live event. The `resolved_size` in the response lets you confirm the system was applied correctly without committing to a full run.

## Webhook payloads

Webhook `order.confirmed` and `order.shipped` payloads now include `resolved_size` on each line. If you parse line items in your webhook handler, add the field to your schema — it is always present in 2.4.0 and later.

```json
{
  "event": "order.confirmed",
  "orderId": "ord_91abc4",
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

## Error reference

| Code | HTTP status | Meaning |
|------|-------------|--------|
| `size_system_ambiguous` | 400 | Multi-facility account sent a bare size label. No `size_system` was present on the line, order root, or account default. |
| `size_system_implicit` | — (warning) | Single-facility account sent a bare size label. Resolved from facility default. Becomes a `400` in **2.6**. |

