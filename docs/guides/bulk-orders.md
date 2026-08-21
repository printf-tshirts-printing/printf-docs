---
title: Bulk orders and templates
section: Guides
last_reviewed: 2026-08-21
owner: platform-docs
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

Bulk orders and saved templates follow the same size resolution rules as single orders. **If you use templates, read the templates section below before 2.6 ships.**

## Bulk orders

Each order in a bulk request is resolved independently. Set `size_system` on each order body, or on each line, so every unit resolves without ambiguity.

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

Each order in the response includes `resolved_size` on every line.

## Saved order templates

⚠️ **Action required before API 2.6.**

Templates are stored snapshots of an order body. They were created before `size_system` existed, so most templates omit it. The resolution rules apply at submission time, not at template creation time.

**What happens today (2.4.0 – 2.5.x):**
- Templates without `size_system` trigger a `size_system_implicit` warning on every submission.
- Single-facility accounts: the order fulfils, but the warning is in the response.
- Multi-facility accounts: the submission is rejected with `400 size_system_ambiguous`.

**What happens in 2.6:**
- `size_system_implicit` becomes `400 size_system_implicit`. All template submissions without `size_system` fail.

### How to fix your templates

1. List your templates via the dashboard or the API.
2. For each template, add `size_system` at the order level and/or on each line.
3. Save the updated template.
4. Resubmit a test order to confirm `resolved_size` looks correct and no warning appears.

Use the `X-Printf-Size-System` response header to verify which system was applied.

