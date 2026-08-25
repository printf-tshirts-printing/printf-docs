---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-25
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/templates/{templateId}, PUT /v2/templates/{templateId}]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Bulk orders and templates

Saved order templates let you store a complete order payload and reuse it across events. Starting in API version 2.4.0, templates must carry an explicit `size_system` on the root and on each line, or orders placed from them will fail with `size_system_ambiguous` on multi-facility accounts.

:::danger
Templates created before 2.4.0 carry no `size_system`. Orders placed from them on multi-facility accounts will fail immediately with `400 size_system_ambiguous`. Update every template before placing orders.
:::

## Creating a template

```json
PUT /v2/templates/tmpl_stackfest_tee
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

Required template fields:

| Field | Required | Notes |
|-------|----------|-------|
| `accountId` | ✅ | Must match the account placing orders from this template |
| `destination` | ✅ | Shipping destination; can be overridden at order time |
| `lines[].designId` | ✅ | Must reference a published design |
| `lines[].garmentSku` | ✅ | Exact SKU; see the catalog API for valid values |
| `lines[].quantity` | ✅ | Default quantity; can be overridden at order time |
| `size_system` | ✅ from 2.4.0 | Template-level default; overridden by line-level value |
| `lines[].size_system` | ✅ from 2.4.0 | Line-level override; takes precedence over template-level |
| `lines[].fit` | recommended | Defaults to garment catalog default when omitted |

## Placing a bulk order from a template

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "templateId": "tmpl_stackfest_tee",
  "size_system": "US",
  "destination": {
    "name": "StackFest Ops",
    "line1": "410 Congress Ave",
    "city": "Austin",
    "region": "TX",
    "postalCode": "78701",
    "countryCode": "US"
  }
}
```

Any field supplied at order time overrides the template value for that order only. The template itself is not modified.

## Response lines in 2.4.0+

Every line in the order response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Verify `resolved_size.system` matches your intent before dispatching to fulfilment. A mismatch here means garments were resolved against a different size system than you expected.

## Updating existing templates for 2.4.0

1. Retrieve each template: `GET /v2/templates/{templateId}`.
2. Add `"size_system": "US"` (or `EU` / `JP`) at the root.
3. Add `"size_system"` and `"fit"` to each line object.
4. Write the update: `PUT /v2/templates/{templateId}`.
5. Place a test order and check `resolved_size` per line.

If you manage templates through any of the client libraries, all five (`printf-js`, `printf-py`, `printf-java`, `printf-go`, `printf-rb`) surface `size_system` and `fit` from version 2.4.0 onward.

## Error codes relevant to bulk orders

| Code | HTTP status | Meaning |
|------|-------------|--------|
| `size_system_ambiguous` | 400 | `size_system` missing and account routes to multiple facilities. Add it to the request or the template. |
| `template_not_found` | 404 | No template with that ID exists on this account. |
| `fit_not_available` | 400 | The `fit` requested is not available for the given `garmentSku`. |
| `quantity_exceeds_limit` | 400 | Line quantity exceeds the bulk limit for your plan. |

For the full sizing and fit reference, including size-system precedence rules, see [Sizing and fit](/guides/sizing).
