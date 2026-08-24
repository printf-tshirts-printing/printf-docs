---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform-docs
covers_endpoints: POST /v2/orders, POST /v2/orders/bulk
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

This guide covers high-volume order submission and the use of saved order templates. If you use templates, read the sizing section carefully — **2.4.0 is a breaking change for templates that omit `size_system`**.

## Bulk submission

Send multiple orders in a single request with `POST /v2/orders/bulk`. Each order in the array is an independent order body and must satisfy the same field requirements as a single `POST /v2/orders` call: `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, `quantity`, `size`, `size_system`, and `fit`.

```json
POST /v2/orders/bulk
{
  "orders": [
    {
      "accountId": "acct_stackfest",
      "size_system": "US",
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
        }
      ]
    }
  ]
}
```

Each order in the response includes its own `lines[].resolved_size` object:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

## Size system in bulk orders

Set `size_system` at the order level to apply it to all lines in that order. You can override it per line. You must resolve ambiguity before submission — any order in a bulk request that would return `400 size_system_ambiguous` causes that individual order to fail. Other orders in the same bulk request are not affected.

## Saved order templates

Templates save a reusable order body that you can hydrate with per-run values (destination, quantity, event name). As of 2.4.0 **every template must include `size_system` at the order level, or `size_system` and `fit` per line**.

### What to do now

1. List your saved templates from the dashboard or your template management tooling. Templates are not visible from the orders API.
2. For each template, open it and verify that `size_system` is set at the order level or on every line.
3. Add `fit` per line if it is not already present.
4. Save the updated template.

Templates that still omit `size_system` will produce a `size_system_implicit` warning on every order they generate (if the account is single-facility) or a `400 size_system_ambiguous` error (if multi-facility). In 2.6, `size_system_implicit` becomes an error and all such templates stop working.

### Before (broken in 2.4 for multi-facility accounts, warning in 2.6 for all):

```json
{
  "accountId": "acct_stackfest",
  "destination": { "..." : "..." },
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### After (correct for 2.4.0 and forward):

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "destination": { "..." : "..." },
  "lines": [
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

## Warnings in bulk responses

Check `warnings` on every order in a bulk response, not just the top-level response envelope. A `size_system_implicit` warning on a single line item means that line is silently using a facility default that disappears in 2.6.

