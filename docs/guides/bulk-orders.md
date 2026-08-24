---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - POST /v2/orders/bulk
  - GET /v2/order-templates
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers high-volume ordering and saved order templates. **If you use saved templates, read the template migration section before upgrading to 2.4.0** — templates bake in field values at save time and will not automatically gain `size_system`.

## Bulk ordering

Send multiple orders in a single call using `POST /v2/orders/bulk`. Each element of the `orders` array is a full order body subject to the same validation as `POST /v2/orders`.

As of **2.4.0**, every order body (including each element in a bulk request) must resolve `size_system` by the time it reaches the fulfilling facility. The fastest way to comply is to set `size_system` at the order level:

```json
POST /v2/orders/bulk
{
  "orders": [
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
          "size": "XL",
          "size_system": "US",
          "fit": "unisex",
          "quantity": 250
        }
      ]
    }
  ]
}
```

In a bulk request, if **any** order in the array triggers `400 size_system_ambiguous`, the entire request is rejected. Fix all orders before resubmitting.

## Saved order templates

Saved templates store a snapshot of the order body at the time you saved them. Templates created before 2.4.0 **do not contain `size_system`**.

### What happens when you use a pre-2.4.0 template today

- **Single-facility account**: the order is accepted with a `size_system_implicit` warning. You will receive the warning in the `warnings` array on the response. **This becomes a `400` error in 2.6.**
- **Multi-facility account**: the order is rejected immediately with `400 size_system_ambiguous`.

### How to update your templates

1. Fetch your templates with `GET /v2/order-templates`.
2. For each template, add `size_system` at the order level and, optionally, `size_system` and `fit` to each line.
3. Save the updated template body.

There is no bulk-update endpoint for templates. Update them one at a time.

### Confirming a template is up to date

After saving, place a test order from the template in a non-production environment. Confirm:

- No `size_system_implicit` warning in the `warnings` array.
- `resolved_size` objects appear on every response line.
- `X-Printf-Size-System` header is present and has the expected value.

## `resolved_size` in bulk responses

The bulk response contains one result per order in the same index order as the request. Each result's lines contain `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For high-volume operations, use `chest_cm` in a post-submission validation step to catch garments that resolved to an unexpected system.

## Warning codes

| Code | Scope | Meaning | Becomes error in |
|---|---|---|---|
| `size_system_implicit` | Single-facility accounts | `size_system` was omitted; facility default was used | 2.6 |

## Error codes

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; `size_system` could not be inferred |

