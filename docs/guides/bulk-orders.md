---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders, POST /v2/order-templates, PATCH /v2/order-templates/{templateId}
covers_sdks: printf-js, printf-py, printf-java, printf-rb, printf-go
---

# Bulk orders and templates

Bulk orders let you place large print runs in a single request. Templates let you reuse order configurations across recurring events. Both are affected by the 2.4.0 size disambiguation change.

## Adding `size_system` to bulk orders

For a bulk run with mixed size systems, set `size_system` per line. For a run where every line shares the same system, set it once at order level and omit it from individual lines.

### Mixed-system bulk order

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
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
      "quantity": 200,
      "size": "XL",
      "size_system": "US",
      "fit": "unisex"
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 50,
      "size": "XL",
      "size_system": "EU",
      "fit": "unisex"
    }
  ]
}
```

The response includes `resolved_size` on every line so you can confirm each one resolved as intended before shipping.

### Single-system bulk order (order-level shorthand)

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "size_system": "JP",
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
      "quantity": 300,
      "size": "XL",
      "fit": "unisex"
    }
  ]
}
```

Every line inherits `JP` from the order. The `resolved_size.chest_cm` for JP XL is 97 cm.

## Templates — action required before 2.6.0

Templates are evaluated every time they generate an order. A template that was created before 2.4.0 will not contain `size_system` and will produce `size_system_implicit` warnings on every order it generates starting now. That warning becomes a hard `400` error in 2.6.0.

**Templates are not visible through the API.** To audit and update them:

1. Open the Printf dashboard.
2. Navigate to **Order templates**.
3. For each template, add `size_system` at the order level, or on each line where the system differs.
4. Save and test with a draft order before your next live run.

This step is easy to forget because templates run silently in the background. If a recurring event order starts failing in 2.6.0 with no code change on your side, an un-updated template is the first place to check.

## Webhook payloads

`order.fulfilled` and `order.updated` webhook events now include `resolved_size` on each line, matching the shape returned by the REST response:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

If your webhook handler maps line objects to a downstream system, add a handler for `resolved_size` before 2.6.0 to avoid silent drops on enriched fields.

## Error and warning reference

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, no `size_system` resolved | Add `size_system` to the order or each line |
| `size_system_implicit` | 200 (warning) | Single-facility account relying on facility default | Add `size_system` to the template and any order bodies |

