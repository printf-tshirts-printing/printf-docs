---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

As of **2.4.0**, every size label must be paired with a size system. A bare `size` value like `"XL"` means different things in different markets — a JP `XL` chest is 97 cm; a US `XL` is 112 cm. The API now enforces an unambiguous system at resolution time.

## Size systems

| `size_system` value | Market | Notes |
|---|---|---|
| `US` | United States / Canada | Default for most North American accounts |
| `EU` | Europe | Numeric ladder (S/M/L labels also accepted) |
| `JP` | Japan | Smaller ladder; XL ≠ US XL |

## Setting `size_system`

You can set `size_system` at three levels. The API resolves it line → order → account → facility default.

1. **Per line** — highest precedence. Use when a single order mixes systems.
2. **Order level** — applies to every line that omits its own `size_system`.
3. **Account default** — set in account settings; applies when neither the order nor the line supplies one.

If none of those three levels supply a value **and** your account can route to more than one facility, the request is rejected with `400 size_system_ambiguous`. If your account routes to a single facility only, the facility's default is used and you receive a `size_system_implicit` warning in the response. **This warning becomes an error in 2.6** — set an explicit value before then.

### Minimal example — order-level system

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
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

### Mixed-system order — per-line override

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
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The second line resolves to a JP XL (97 cm chest), not a US XL (112 cm chest), because the per-line `size_system` overrides the order-level value.

## The `fit` field

`fit` is set per line. Accepted values include `unisex` (the most common). Check your account's garment catalog for the full list of fits available on a given `garmentSku`.

## The `resolved_size` response field

Every line in the response now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

`resolved_size` also appears in webhook payloads. Use `chest_cm` to verify that the garment dimension matches your expectation before the order enters production.

## The `X-Printf-Size-System` response header

The `X-Printf-Size-System` response header reflects the system that was used to resolve all lines when a single system applies to the full order. When lines resolve under different systems, the header is omitted.

## Error reference

| Code | HTTP status | Meaning | Action |
|---|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` supplied; account routes to multiple facilities | Add `size_system` to the order or each line |
| `size_system_implicit` | — (warning) | No `size_system` supplied; facility default used | Add `size_system` before 2.6, when this becomes an error |

## Saved order templates

Order templates do not include `size_system` unless you added it. Any template created before 2.4.0 will either receive the `size_system_implicit` warning (single-facility accounts) or be rejected with `size_system_ambiguous` (multi-facility accounts) at submission time. **Open your templates and add `size_system` explicitly.**

