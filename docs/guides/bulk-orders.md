---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-21
owner: product
covers_endpoints: ["POST /v2/orders"]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Bulk orders and templates

## Bulk orders

Bulk orders use the same `POST /v2/orders` shape as single orders, repeated across a batch payload. As of **2.4.0**, every order in the batch must follow the same `size_system` rules as single orders.

### Required fields per order in a batch

| Field | Level | Notes |
|---|---|---|
| `accountId` | Order | Must match the authenticated account |
| `destination` | Order | Full address object required on each order |
| `designId` | Line | Per line |
| `garmentSku` | Line | Per line |
| `quantity` | Line | Per line |

Add `size_system` at the order level and/or per line. A batch where any order omits `size_system` on a multi-facility account will have that order rejected with `400 size_system_ambiguous`. The rest of the batch is unaffected — Printf validates each order independently.

```json
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
```

## Saved order templates

> **Action required before 2.6:** Templates saved before 2.4.0 do not contain `size_system`. Submitting a stale template produces a `size_system_implicit` warning today and will produce a `400` error in **2.6**.

Templates are not enumerable through the API. To find and update them:

1. Open the Printf dashboard → **Templates**.
2. For each template, open the editor and add `size_system` at the order level and on every line.
3. Save. The next submission from that template will include the field and will not emit a warning.

If you manage templates programmatically via export/import, update the JSON before re-importing:

**Before (2.3 template fragment):**
```json
{
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

**After (2.4+ template fragment):**
```json
{
  "size_system": "US",
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

Note that `accountId` and `destination` are stored in the template header, not the lines fragment — your export format may vary.

## Webhooks and bulk fulfillment events

Fulfillment webhooks for bulk orders now include `resolved_size` on every line item. See [Webhooks](webhooks.md) for the updated payload shape.

