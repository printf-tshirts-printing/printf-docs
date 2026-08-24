---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: dx
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

This guide covers submitting multiple orders in a single call and managing saved order templates.
As of **2.4.0**, both bulk payloads and saved templates must include `size_system`.

## Bulk order payload

Each order in a bulk request is structurally identical to a single `POST /v2/orders` body.
`accountId`, `destination`, `designId`, `garmentSku`, and `quantity` are required on every order.
Add `size_system` at the order level, the line level, or both.

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
          "size": "XL",
          "size_system": "US",
          "fit": "unisex",
          "quantity": 250,
          "garmentSku": "tee-classic-black"
        },
        {
          "designId": "dsn_7fa91c",
          "size": "L",
          "size_system": "US",
          "fit": "unisex",
          "quantity": 100,
          "garmentSku": "tee-classic-black"
        }
      ]
    }
  ]
}
```

## Saved order templates

> **Action required if you use saved templates.** Templates are not visible through the API — they
> are stored configurations you may have set up in the dashboard or via support. If a template was
> created before 2.4.0, it does not contain `size_system` and will behave differently depending on
> your account type:
>
> - **Multi-facility account**: orders submitted from the template are rejected with `400 size_system_ambiguous`.
> - **Single-facility account**: orders submitted from the template succeed with a `size_system_implicit` warning until **2.6**, at which point they are also rejected.
>
> Edit each affected template to add `size_system` at the order level before upgrading to 2.6.

### How to identify affected templates

1. Go to **Dashboard → Templates**.
2. Open each template and check whether `size_system` is set at the order or line level.
3. Add `size_system` to any template that lacks it.
4. Save and test with a one-unit order before your next event run.

## `resolved_size` in bulk responses

Every line in a bulk response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Verify this field in staging against each template before a live event, particularly when mixing
US and EU attendee populations across lines.

## Error handling in bulk requests

A `size_system_ambiguous` error on any line causes the **entire bulk request** to be rejected.
Fix the offending line(s) and resubmit the full payload — partial application does not occur.

