---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-21
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

**As of order-api 2.4.0, `size_system` is required on every line (or at the order level as a default).** Saved templates that omit it will produce `400 size_system_ambiguous` for multi-facility accounts or `size_system_implicit` warnings for single-facility accounts. Review all templates before your next submission.

## Bulk order structure

A bulk order is a single `POST /v2/orders` call with multiple entries in the `lines` array. Each line specifies its own design, size, fit, quantity, and garment.

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
      "quantity": 80,
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

Set `size_system` at the order level as a default and again per line for clarity. The line-level value always wins if both are present.

## Using saved order templates

Templates let you store a reusable order skeleton — destination, lines, and defaults — and submit it by reference. They are not visible through the catalog API; you manage them through the dashboard or by saving the response from a successful order.

### Templates and size_system (action required)

**Templates saved before 2.4.0 do not include `size_system`.** When you submit a template against a multi-facility account, the API cannot resolve the size system from routing and returns `400 size_system_ambiguous`. For single-facility accounts, it returns a `size_system_implicit` warning on every submission — this becomes a hard error in **2.6**.

To fix a saved template, open it in the dashboard and add `size_system` to the order level and to each line, or delete and recreate it using the updated payload shape above.

### Verifying `resolved_size` after submission

Every submitted line now returns a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk orders, check `resolved_size.chest_cm` on every line before accepting the proof. A JP `XL` is 97 cm; a US `XL` is 112 cm — a 15 cm difference that will produce the wrong garment if the system was resolved by fallback rather than by your explicit field.

## Multi-facility accounts

If your account can route to more than one facility, you must set `size_system` explicitly on every order or line. The API will not guess based on routing, and omitting the field returns `400 size_system_ambiguous` immediately.

To check which facilities your account can route to, inspect the `facilities` array on your account object. If it contains more than one entry, you are a multi-facility account.

## Error reference

| Error code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, `size_system` absent, cannot resolve from routing | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account, `size_system` absent, resolved from facility default | Add `size_system` before 2.6 when this becomes a 400 |
