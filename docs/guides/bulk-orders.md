---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders, POST /v2/orders/bulk, GET /v2/templates
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

This guide covers high-volume ordering and saved order templates. Both are affected by the 2.4.0 size disambiguation change.

## Saved templates and 2.4.0

**Saved order templates do not automatically inherit the new fields.** If a template was created before 2.4.0 it will not contain `size_system` or `fit`. When that template is submitted:

- Single-facility accounts: the request succeeds with a `size_system_implicit` warning.
- Multi-facility accounts: the request is **rejected with `400 size_system_ambiguous`**.

Audit every saved template before 2.6. Retrieve each template via `GET /v2/templates`, check whether `size_system` is present on the order body and on each line, and re-save with the field added. There is no bulk migration endpoint.

## Bulk order requests

Each order object within a bulk request is validated independently. A single line missing `size_system` in a multi-facility context will reject that individual order; the rest of the batch continues processing.

### Bulk request example

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
        }
      ]
    }
  ]
}
```

## Verifying resolved sizes in bulk responses

Each order line in the response includes `resolved_size`. For bulk submissions processing large catalogs, diff the `resolved_size.chest_cm` values against your garment spec sheet before releasing to production. A system mismatch (e.g. sending `JP` sizes but targeting a US facility) will resolve without error but produce the wrong garment dimensions.

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

## The fit field in bulk orders

Set `fit` per line. Valid values: `unisex`, `mens`, `womens`. When omitted, `unisex` is assumed. For events with mixed audiences, specify `fit` explicitly on each line rather than relying on the default.

## Error handling for bulk batches

| Code | HTTP status | Scope | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Per order | Add `size_system` to that order or its lines |
| `size_system_implicit` | 200 warning | Per order | Add `size_system` before 2.6 |

