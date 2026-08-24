---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, POST /v2/orders/bulk
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

## Saved order templates and 2.4.0

> **Action required if you use saved order templates.** Templates store the request body verbatim. Any template created before 2.4.0 that does not include `size_system` will trigger `size_system_implicit` warnings now and **`400` errors in 2.6**. Open every template in the dashboard and add `size_system` at the order or line level before upgrading to 2.6.

## Sending a bulk order

Bulk orders submit multiple lines in a single request. All lines in the same request share the top-level `size_system` unless a line overrides it.

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
      "size": "M",
      "size_system": "EU",
      "fit": "mens",
      "quantity": 75,
      "garmentSku": "hoodie-zip-navy"
    }
  ]
}
```

The third line overrides `size_system` to `EU` because `hoodie-zip-navy` is sourced from a EU facility. Each line's `resolved_size` in the response tells you exactly what was applied.

## Mixing size systems in one order

Set `size_system` on any line that deviates from the order-level default. You do not need to repeat the order-level value on every line.

```json
{ "designId": "dsn_abc", "size": "L", "size_system": "JP", "fit": "unisex", "quantity": 50, "garmentSku": "tee-slim-grey" }
```

A JP `L` resolves to a different chest measurement than a US `L`. Confirm `resolved_size.chest_cm` before finalising bulk runs.

## Multi-facility accounts and `size_system_ambiguous`

If your account routes to more than one facility and a line has no resolvable `size_system`, the API returns:

```
HTTP 400 size_system_ambiguous
```

This is not retryable. Add `size_system` to the offending line or to the order body.

## Webhook payloads

`resolved_size` is included on every line in webhook payloads from 2.4.0 onwards. If you process webhook bodies with a strict schema, add the field before enabling 2.4.0 webhooks.

## Template checklist before 2.6

- [ ] Open every saved template in the dashboard.
- [ ] Confirm `size_system` is present at the order level, or on every line.
- [ ] Re-save the template.
- [ ] Place a test order from the updated template in the staging environment and verify `resolved_size` on each line.
- [ ] Repeat for all accounts that use the template.
