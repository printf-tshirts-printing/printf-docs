---
title: Quickstart
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

# Quickstart

This guide gets you from zero to a confirmed print order in under ten minutes. It reflects order-api **2.4.0** and later. If you are on an earlier version, upgrade before following these steps — `size_system` is required for multi-facility accounts and will be required for all accounts in 2.6.

## Prerequisites

- An API key from the Printf dashboard
- Your `accountId` (format: `acct_…`)
- Your `facilityId` (format: `fac-…`)
- At least one design uploaded and confirmed (`designId`, format: `dsn_…`)
- A `garmentSku` from your approved garment catalog

## Step 1 — Place your first order

Send a `POST /v2/orders` with all required fields. The minimum viable request for 2.4.0 includes `accountId`, `destination`, and at least one line with `designId`, `garmentSku`, `quantity`, `size`, and `size_system`.

```json
POST /v2/orders
Authorization: Bearer <your_api_key>
Content-Type: application/json

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

Required fields on every request: `accountId`, `destination`, `designId`, `garmentSku`, `quantity`. Missing any of these returns `400`.

## Step 2 — Read the response

A successful response is `201 Created`. Each line in the response body includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Check the `X-Printf-Size-System` response header to confirm which size system the API applied. If it reads `mixed`, individual lines were resolved under different systems — inspect each line's `resolved_size.system`.

## Step 3 — Handle warnings

If the response body contains a `size_system_implicit` warning, your account is resolving size system from the facility default rather than from your payload. **This works today but becomes a `400` error in 2.6.** Add `size_system` to your order or line now.

```json
{
  "warnings": [
    {
      "code": "size_system_implicit",
      "message": "size_system resolved from facility default; will become an error in 2.6"
    }
  ]
}
```

## Step 4 — Choose your SDK

All five official clients are updated for 2.4.0 with typed `size_system`, `fit`, and `resolved_size` fields.

| SDK | Package |
|---|---|
| JavaScript / TypeScript | `printf-js` |
| Python | `printf-py` |
| Java | `printf-java` |
| Go | `printf-go` |
| Ruby | `printf-rb` |

Install the `2.4.x` patch release of the relevant package. Earlier releases do not have `size_system` or `fit` in their type definitions.

## Common errors

| Code | HTTP status | Fix |
|---|---|---|
| `size_system_ambiguous` | 400 | Your account routes to multiple facilities. Add `size_system` at the order or line level. |
| `size_system_implicit` | warning | Add `size_system` before 2.6 to avoid a breaking error. |
| `size_not_found` | 422 | The `size` label does not exist in the specified `size_system`. Check your garment catalog. |

