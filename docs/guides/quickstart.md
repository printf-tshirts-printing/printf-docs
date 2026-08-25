---
title: Quickstart
section: guides
last_reviewed: 2026-08-25
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Quickstart

This guide gets you from zero to a successfully placed order in under ten minutes. It targets API version 2.4.0. If you are on an earlier version, the request shape is different — see the [migration note](/guides/sizing#migrating-to-240).

## Prerequisites

- An API key. Find it in [your dashboard](https://app.printf.dev/settings/api-keys).
- Your `accountId`. Shown on the account overview page and in every order response.
- A published design. Use your `designId` from the design editor, or the value below for the sandbox.

## 1. Place your first order

Replace the values in angle brackets with your own, then run:

```bash
curl -X POST https://api.printf.dev/v2/orders \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "<YOUR_ACCOUNT_ID>",
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
        "quantity": 1,
        "garmentSku": "tee-classic-black"
      }
    ]
  }'
```

A `200` response confirms the order was accepted. A `400 size_system_ambiguous` means your account routes to more than one facility and you must include `size_system` — which the example above already does.

## 2. Read `resolved_size` from the response

Every line in the response includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Confirm that `resolved_size.system` matches what you sent. If it differs, a line-level `size_system` override was not applied as you expected.

## 3. Install a client library (optional)

All five official libraries support `size_system` and `fit` from version 2.4.0:

| Language | Install |
|----------|---------|
| JavaScript | `npm install printf-js@2.4.0` |
| Python | `pip install printf-py==2.4.0` |
| Java | `implementation 'dev.printf:printf-java:2.4.0'` |
| Go | `go get dev.printf/printf-go@v2.4.0` |
| Ruby | `gem install printf-rb -v 2.4.0` |

## Required fields reference

Every `POST /v2/orders` request must include these fields, whether you use a template or not:

| Field | Type | Notes |
|-------|------|-------|
| `accountId` | string | Your account identifier |
| `destination` | object | Full shipping address including `countryCode` |
| `lines[].designId` | string | Published design identifier |
| `lines[].garmentSku` | string | Exact garment SKU from the catalog |
| `lines[].quantity` | integer | Units to produce; minimum 1 |
| `size_system` | string | `US`, `EU`, or `JP`. Required now for multi-facility accounts; required for all accounts in 2.5.0. |

## Next steps

- [Sizing and fit](/guides/sizing) — size systems, `fit` values, and how `resolved_size` is calculated.
- [Bulk orders and templates](/guides/bulk-orders) — save and reuse order configurations across events.
- [API reference](/api/orders) — full field definitions and every error code.
