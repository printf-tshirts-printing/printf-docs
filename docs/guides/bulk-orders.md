---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Bulk orders and templates

This page covers sending large order volumes and managing saved order templates. If you use templates, read the [sizing migration note](/guides/sizing#migrating-saved-templates) before your next deployment — saved templates created before 2.4.0 are missing a now-required field for multi-facility accounts.

## Sending bulk orders

For volumes above a few hundred units, send lines in a single request rather than one request per line. The API accepts up to 500 lines per order.

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
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "quantity": 90,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "quantity": 60,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Each line in the response carries a `resolved_size` object:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

If a line omits `size_system`, it inherits the order-level value. The `X-Printf-Size-System` response header shows which system was applied to the order.

## Mixing size systems in one order

If an event ships internationally, you may need EU and US sizes in the same order. Override `size_system` at the line level:

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
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "EU",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Note that `XL` in `US` (112 cm chest) and `XL` in `EU` (107 cm chest) cut differently. Confirm the intended system with your fulfilment contact before placing a mixed-system order for the first time.

## Saved templates and 2.4.0 {#templates-and-240}

A saved order template is a stored request body you submit by reference. Templates created before 2.4.0 contain no `size_system`.

**If your account routes to more than one facility**, submitting an old template without modification returns `400 size_system_ambiguous`. The order is not created.

**If your account routes to a single facility**, the order is accepted but the response includes a `size_system_implicit` warning. This warning becomes a `400` error in **2.6**.

To update a template, retrieve it, add `"size_system"` at the root, and save it back. The [sizing guide](/guides/sizing#migrating-saved-templates) has the full diff.

## Webhook payloads

The `order.confirmed` and `order.shipped` webhook events now include `resolved_size` on each line. If your webhook consumer writes line data to a database, add a column or field for `resolved_size` before enabling 2.4.0 webhooks, or the field will be silently dropped.
