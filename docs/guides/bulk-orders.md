---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-21
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

This guide covers submitting large orders and using saved order templates. Both are affected by the `size_system` and `fit` changes introduced in **2.4.0**.

## Submitting a bulk order

Bulk orders follow the same `POST /v2/orders` shape as single orders. Include `size_system` at the order level to apply it across all lines, and override per line when a line needs a different system.

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
      "quantity": 120,
      "garmentSku": "tee-classic-black"
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
      "quantity": 80,
      "garmentSku": "tee-fitted-white"
    }
  ]
}
```

Each response line includes `resolved_size` so you can confirm every garment resolved to the chest measurement you intended before the order enters production.

## Saved order templates

> ⚠️ **Templates are affected by the 2.4.0 changes.** Templates that existed before 2.4.0 do not have `size_system` or `fit` stored in them. When a template is used to submit an order, bare size labels fall through the resolution chain (line → order → account → facility default). If your account routes to more than one facility, the submission will be rejected with `400 size_system_ambiguous`.

**Before you use any existing template**, open it and confirm whether it stores explicit `size_system` values. If it does not, edit the template to add `size_system` to the order root and `size_system` + `fit` to every line before the next submission.

Templates are not visible in the orders API — you manage them in the dashboard under **Account → Templates**. The API does not return a list of saved templates, so a bulk-submit pipeline that iterates saved templates must be audited manually.

### Template checklist for 2.4.0

- [ ] Every line in every template has an explicit `size_system`
- [ ] Every line in every template has an explicit `fit` where the garment supports it
- [ ] Order root has `size_system` as a fallback even if every line sets it
- [ ] You have verified `resolved_size.chest_cm` in staging for at least one line of each template

## Handling `resolved_size` in bulk responses

For bulk orders, `resolved_size` appears on every line in the response:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

If any line shows a `system` or `chest_cm` you did not expect, cancel the order before it reaches the print queue. There is no in-flight size correction endpoint.

## Error reference

| Code | Meaning | Fix |
|---|---|---|
| `size_system_ambiguous` | Account routes to multiple facilities; no `size_system` resolved | Add explicit `size_system` to the order or each line |
| `size_system_implicit` | Single-facility account; `size_system` fell through to facility default | Warning in 2.4.0, error in 2.6.0 — add explicit `size_system` now |

