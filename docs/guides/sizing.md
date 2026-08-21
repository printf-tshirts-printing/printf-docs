---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

Starting in **order-api 2.4.0**, size labels such as `"L"` and `"XL"` always resolve to a specific measurement. You control resolution by supplying `size_system` at the order level, the line level, or both. If you omit it entirely, the API falls back through a defined chain — and may reject the request.

## Size systems

| System | Key | XL chest (cm) |
|---|---|---|
| United States | `US` | 112 |
| European | `EU` | 106 |
| Japan | `JP` | 97 |

A JP `XL` and a US `XL` are not the same garment. Sending the wrong system produces the wrong item — there is no warning after fulfillment.

## Setting size_system

You can set `size_system` at two levels. The line-level value takes precedence over the order-level value.

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

If you set `size_system` on the order but not the line, all lines inherit the order value. If you set it on the line, only that line uses the override.

## The fit field

`fit` is set per line. Accepted values depend on the garment SKU — check the product catalog for what each SKU supports. Common values: `unisex`, `womens`, `mens`.

## resolved_size on responses

Every response line and every webhook payload now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` to verify the garment is what you intended. Mismatches between your expected measurement and the resolved value indicate a `size_system` misconfiguration.

## X-Printf-Size-System response header

Every response carries the header `X-Printf-Size-System` with the system that was applied to the order. Useful for logging and debugging without parsing the body.

## Fallback resolution order

When `size_system` is absent, the API resolves it in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default

Step 4 depends on routing, not on your payload.

**Accounts that can route to more than one facility:** step 4 is ambiguous. The request is rejected with `400 size_system_ambiguous`. You must supply `size_system` explicitly.

**Single-facility accounts:** step 4 succeeds, but the response carries a `size_system_implicit` warning. This warning becomes an error in **2.6**. Add `size_system` now.

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities; `size_system` is required |
| `size_system_implicit` | — (warning) | Size system resolved from facility default; will error in 2.6 |
