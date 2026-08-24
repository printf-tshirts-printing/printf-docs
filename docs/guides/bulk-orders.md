---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
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

Bulk orders and saved templates follow the same field rules as single orders. As of **2.4.0**, that includes the `size_system` requirement.

## ⚠️ Template owners: action required before 2.6

Saved order templates are not visible in the API response for `GET /v2/orders` and are easy to overlook. If a template was saved before 2.4.0 it almost certainly lacks `size_system`. Those templates will:

- **Now (2.4.0):** produce a `size_system_implicit` warning on every order created from the template.
- **In 2.6:** be rejected with `400 size_system_ambiguous` or `400 size_system_implicit` (promoted to error).

Audit your templates now. Add `size_system` at the order level (or per line if lines mix systems).

## Sending a bulk order

Each order in a bulk request is an independent object and must carry its own `size_system`, `accountId`, `destination`, and required line fields.

```json
POST /v2/orders/bulk
[
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
```

## Mixed size systems in a bulk request

Different orders in the same bulk request can use different size systems — set `size_system` independently on each order object, or per line within each order.

| Scenario | Recommended approach |
|---|---|
| All lines same system | Set `size_system` once at the order level |
| Lines mix US and EU | Set order-level to majority, override with line-level |
| Lines mix US and JP | Set explicitly per line — the 15 cm chest difference is too large to risk implicit resolution |

## Reading `resolved_size` in bulk responses

Each line in the bulk response carries a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Validate `system` and `chest_cm` against your order intent before treating the batch as accepted. A mismatch here means the size resolved differently than you expected — correct and resubmit.

## Error reference

| Code | Status | Meaning |
|---|---|---|
| `size_system_ambiguous` | `400` | Account routes to multiple facilities; `size_system` cannot be inferred. Set it explicitly. |
| `size_system_implicit` | warning → `400` in 2.6 | `size_system` was inferred from account or facility default. Add it explicitly. |

## SDK notes

All five client libraries (printf-js, printf-py, printf-java, printf-go, printf-rb) have been updated for 2.4.0. Upgrade to the 2.4.x release of your library to get typed `size_system`, `fit`, and `resolved_size` fields. Earlier library versions will send and receive these fields as untyped strings or generic maps.
