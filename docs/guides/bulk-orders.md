---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
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

This guide covers large-quantity orders and saved order templates. If you use templates, read the section on [templates and size_system](#templates-and-size_system) — templates are the most common source of `size_system_ambiguous` errors after upgrading to 2.4.0.

## Sending a bulk order

Bulk orders use the same `POST /v2/orders` endpoint as single orders. There is no separate bulk endpoint. Include all lines in the `lines` array in one request.

Required fields on every request: `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, and `quantity`.

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
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 150,
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

Set `size_system` at the order level when all lines share a system. Override it per line when a single order mixes systems (uncommon, but valid).

## Templates and size_system

**Templates do not automatically inherit `size_system`.** If you have saved order templates that were created before 2.4.0, they contain bare `size` labels with no `size_system`. When you submit a template-derived order:

- **Single-facility account:** the order is accepted with a `size_system_implicit` warning. The warning becomes an error in 2.6.
- **Multi-facility account:** the order is rejected immediately with `400 size_system_ambiguous`.

To fix a template, add `size_system` at the order level, and optionally per line. You cannot patch a saved template via the API — you must re-save it with the new fields, or inject `size_system` at submission time in your integration code.

## `resolved_size` in bulk responses

Every line in the response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk orders, iterate all lines and verify `resolved_size.system` matches your intent before treating the order as confirmed. A mismatch here means garments will be cut to the wrong ladder.

## Error codes relevant to bulk orders

| Code | HTTP | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` and account routes to multiple facilities | Add `size_system` to the request or to the template |
| `size_system_implicit` | 200 (warning) | No `size_system`; resolved from single-facility default | Add `size_system` before 2.6 to avoid a hard error |

## Webhook payloads for bulk orders

Webhook payloads for bulk orders include `resolved_size` on each line, identical to the REST response. If your webhook consumer parses line items, update it to accept (and ideally store) the new `resolved_size` object.
