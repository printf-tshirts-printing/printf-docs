---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of **2.4.0**, every size label must be accompanied by a size system. A bare `"XL"` means different things across markets — a JP `XL` has a 97 cm chest; a US `XL` has a 112 cm chest. Omitting the system and letting the API guess is no longer supported for multi-facility accounts, and is deprecated for single-facility accounts.

## Size systems

| Value | Market | Example chest measurement at XL |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 104 cm |
| `JP` | Japan | 97 cm |

## Where to set `size_system`

`size_system` can be set at two levels. The more specific value wins.

| Level | Field | Scope |
|---|---|---|
| Order body | `size_system` | Default for every line in the order |
| Order line | `size_system` | Overrides the order-level default for that line only |

Setting it once at the order level is the recommended pattern when all lines share the same system.

## The `fit` field

`fit` is a per-line field. Accepted values depend on the garment; common values are `unisex`, `womens`, and `mens`. When omitted, the garment's default fit is used.

## `resolved_size` on responses

Every line in the response and in webhook payloads now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size` to confirm the exact interpretation before printing. A mismatch here — not at fulfillment time — is the right place to catch sizing errors.

## `X-Printf-Size-System` response header

The response includes an `X-Printf-Size-System` header reflecting the effective size system resolved for the order. Useful for logging and debugging without parsing the body.

## Full example

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

Response line:

```json
{
  "designId": "dsn_7fa91c",
  "garmentSku": "tee-classic-black",
  "quantity": 250,
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

## Error: `size_system_ambiguous`

If you omit `size_system` at both levels and your account is configured to route to more than one facility, you will receive:

```
HTTP 400 size_system_ambiguous
```

Fix: add `size_system` at the order body level, the line level, or both. See the example above.

## Warning: `size_system_implicit`

If you omit `size_system` and your account routes to exactly one facility, the API resolves against that facility's default and returns a `size_system_implicit` warning. **This warning becomes an error in 2.6.** Add `size_system` now to avoid a breaking change at upgrade.

## Saved order templates

Templates do not expose `size_system` through the API, but they are evaluated at submission time using the same rules. If you have saved templates, open them and add `size_system` to the template body or to each line. A template without `size_system` will start returning `size_system_ambiguous` (multi-facility accounts) or `size_system_implicit` warnings (single-facility accounts) immediately after upgrading to 2.4.0.

