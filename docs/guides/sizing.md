---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
  - printf-py
  - printf-go
  - printf-java
  - printf-rb
---

# Sizing and fit

As of order-api **2.4.0**, every order resolves to an explicit size system. A `US` `XL` is a 112 cm chest; a `JP` `XL` is 97 cm. The API rejects ambiguous requests rather than guessing.

> **SDK note:** `printf-js`, `printf-py`, `printf-go`, `printf-java`, and `printf-rb` do not yet model `size_system`, `fit`, or `resolved_size`. Use the raw HTTP API or pass extra fields as untyped maps until SDK releases ship.

## Size systems

| System | Code | Example chest (XL) |
|---|---|---|
| United States | `US` | 112 cm |
| European | `EU` | 107 cm |
| Japanese | `JP` | 97 cm |

Set `size_system` once at the order level to apply to all lines, then override it per line when a single order mixes systems.

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
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

## Fit

Supply `fit` on each line item. Accepted values: `unisex`, `womens`, `mens`. When omitted the facility default applies, and a `size_system_implicit` warning is returned.

## resolved_size

Every line in the response and in webhook payloads includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` for any downstream logic that compares sizes across systems. Do not compare `label` values from different systems — `JP XL` and `US XL` share a label but differ by 15 cm.

## Resolution order

When `size_system` is absent the API resolves in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default

Step 4 depends on routing. If your account can route to more than one facility, step 4 is ambiguous and the request is rejected with `400 size_system_ambiguous`. Add `size_system` explicitly to avoid this.

## Response header

All order responses include `X-Printf-Size-System` set to the system applied to the order.

## Error and warning codes

| Code | HTTP status | Description |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` supplied; account routes to multiple facilities. Fix: add `size_system`. |
| `size_system_implicit` | warning (200) | No `size_system` supplied; single-facility account. Will become `400` in 2.6. |

## Saved templates

Templates are not validated at save time. If a saved template omits `size_system`, orders placed from that template produce `size_system_implicit` warnings today and fail with `size_system_ambiguous` or a hard error in 2.6. Update every template that does not include an explicit `size_system`.

