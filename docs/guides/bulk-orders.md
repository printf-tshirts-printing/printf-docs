---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform-docs
covers_endpoints:
  - POST /v2/orders
  - POST /v2/orders/bulk
  - GET /v2/order-templates/{templateId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Bulk orders and templates

This guide covers placing large orders, batching lines for multi-SKU events, and managing saved order templates. As of **order-api 2.4**, every order and template must include an explicit `size_system`. Templates that were saved before 2.4 without `size_system` will fail on submission against multi-facility accounts.

## Placing a bulk order

Send multiple lines in a single request. Each line must include `designId`, `garmentSku`, `size`, `quantity`, and (as of 2.4) `size_system` or a fallback at the order level.

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
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 300,
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

Setting `size_system` at the order level applies it to all lines. You can override it per line when a single order spans multiple markets.

## Mixed-system orders

If your event distributes garments in multiple size markets — for example staff in the US and sponsors in Japan — set `size_system` per line and omit the order-level field, or set it as a default and override the exceptions:

```json
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
    "size": "XL",
    "size_system": "JP",
    "fit": "unisex",
    "quantity": 60,
    "garmentSku": "tee-classic-black"
  }
]
```

A JP `XL` resolves to 97 cm chest; a US `XL` resolves to 112 cm. Both lines are `XL` but they are different garments — confirm `resolved_size.chest_cm` in the response.

## Saved order templates

> **Action required before 2.6.** Templates saved without `size_system` will produce a `size_system_implicit` warning on single-facility accounts now, and will fail with a hard error on multi-facility accounts immediately. Update templates before order-api 2.6 ships.

Order templates let you save a partially or fully specified order and submit it on demand. Templates are not visible through the standard orders list — retrieve them explicitly:

```
GET /v2/order-templates/{templateId}
```

### Auditing your templates for 2.4 compliance

1. List all templates: `GET /v2/order-templates`
2. For each template, check that `size_system` is present at the order level, or on every line.
3. If missing, update the template with `PATCH /v2/order-templates/{templateId}` and add `size_system` where needed.
4. Re-submit a test order from the updated template and confirm `resolved_size` is present on every line in the response.

### Template example (2.4-compliant)

```json
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
    }
  ]
}
```

## Error reference

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account can route to multiple facilities and no `size_system` was given | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | Single-facility account resolved size system from facility default | Add `size_system` before 2.6 |

## `resolved_size` in bulk responses

Every line in the response carries `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk orders use `resolved_size` to build packing lists and size-run reports. Do not reconstruct measurements from label strings.
