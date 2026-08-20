---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-20
owner: docs-platform
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
---

# Bulk orders and templates

> **Action required before 2.6.0.** Saved order templates that do not include `size_system` will start failing with `400` errors in release 2.6.0. Read the migration section below before that release ships.

> **SDK notice:** `printf-js` and `printf-py` both report `supportsSizeSystem=false`. They do not yet model `size_system`, `fit`, or `resolved_size`. Use raw HTTP for bulk workflows that need the new size fields until updated SDK releases are available.

## Bulk order requests

Bulk workflows call `POST /v2/orders` in a loop or via the batch helper. Each request must include `accountId`, `destination`, `designId`, `garmentSku`, and `quantity` on every line.

As of **2.4.0**, each request should also include `size_system`. A complete line item looks like this:

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

## Saved templates — highest-risk cohort for this release

Saved order templates are stored copies of request payloads that you replay without editing. Because they were created before 2.4.0, they do not contain `size_system` or `fit`.

When a saved template is replayed:

- **Single-facility accounts** — the request succeeds today, but the response includes a `size_system_implicit` deprecation warning. **This becomes a `400` error in 2.6.0.**
- **Multi-facility accounts** — the request fails **immediately** with `400 size_system_ambiguous`. Templates break as of 2.4.0, not 2.6.0.

Templates are not visible through the API. You must audit them in the dashboard under **Settings → Order Templates**, or retrieve them through your internal tooling if you manage templates programmatically.

## How to update a saved template

1. Open the template in **Settings → Order Templates**.
2. Add `size_system` at the order level matching the facility you route to:
   - `fac-atx` (Austin) → `"US"`
   - `fac-ber` (Berlin) → `"EU"`
   - `fac-osa` (Osaka) → `"JP"`
3. Optionally add `size_system` and `fit` per line for precision.
4. Save and run a test order to confirm `resolved_size.chest_cm` matches your expected garment dimensions.

## Verifying resolved size in bulk runs

Every order response carries `resolved_size` per line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Log `resolved_size.chest_cm` from your bulk run output and diff it against your expected values before releasing a new template to production. A JP `XL` (97 cm) versus a US `XL` (112 cm) difference is 15 cm — large enough to produce the wrong garment tier.

## Error reference

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account, bare size label, facility default cannot be determined | Add `size_system` to the request or template |
| `size_system_implicit` | — (warning) | Single-facility account resolved to facility default | Add `size_system` before 2.6.0 |

