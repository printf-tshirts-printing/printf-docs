---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: integrations
covers_endpoints: POST /v2/orders, GET /v2/orders/{id}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

As of 2.4.0, every order line requires an explicit size system. Bare size labels are no longer resolved by guess.

## Size systems

| System | Field value | Notes |
|---|---|---|
| United States | `US` | Default for most North American facilities |
| European | `EU` | Used by EU and UK facilities |
| Japanese | `JP` | Used by APAC facilities |

Size ladders differ **materially** across systems. Always confirm which system your fulfillment facility uses before placing production volume.

| Label | US chest (cm) | JP chest (cm) |
|---|---|---|
| M | 100 | 92 |
| L | 106 | 96 |
| XL | 112 | 97 |

## Declaring the size system

You can set `size_system` at two levels:

1. **Per line** — highest priority, overrides everything.
2. **Order level** — applies to every line that does not set its own `size_system`.

If neither is present the API falls back to the account default, and then to the fulfilling facility's default. That last step depends on routing. Accounts that can route to **more than one facility** are rejected with `400 size_system_ambiguous` — the API will not guess on your behalf. Single-facility accounts receive a `size_system_implicit` warning instead; that warning becomes a hard error in 2.6.

**Recommendation:** always set `size_system` explicitly on every request.

## The `fit` field

`fit` is set per line and controls the garment cut.

| Value | Description |
|---|---|
| `unisex` | Straight cut, default when omitted |
| `mens` | Tapered at shoulders |
| `womens` | Tapered at waist |

## Request example

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

## The `resolved_size` response field

Every line in the response now includes `resolved_size`, showing exactly what the fulfillment system interpreted:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header also reflects the system that was applied at the order level.

## Error and warning codes

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account sent a bare size with no resolvable system | Add `size_system` to the request |
| `size_system_implicit` | 200 (warning) | Single-facility account relying on facility default | Add `size_system` before 2.6 |

