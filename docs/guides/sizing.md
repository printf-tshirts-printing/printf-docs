---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform-docs
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

As of **order-api 2.4**, every size label must be anchored to a size system. A bare `"XL"` means different things in different markets — a JP `XL` is 97 cm chest, a US `XL` is 112 cm. Getting the system wrong means the wrong garment ships.

## Size systems

| Value | Market |
|---|---|
| `US` | United States / Canada |
| `EU` | Europe |
| `JP` | Japan |

## Chest measurements by system

| Label | US (cm) | EU (cm) | JP (cm) |
|---|---|---|---|
| M | 99 | 98 | 91 |
| L | 107 | 104 | 97 |
| XL | 112 | 110 | 97 |
| XXL | 117 | 116 | 102 |

JP ladders are compressed at the top end. If you fulfil events in both the US and Japan from a single template, set `size_system` per line.

## Setting the size system

`size_system` can appear at two levels. The line value takes priority; if absent the order-level value is used.

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

## The `fit` field

Each line now accepts a `fit` value. Supported values depend on the garment SKU — check the catalogue endpoint for the SKU you are ordering. Common values: `unisex`, `womens`, `mens`.

Fit affects the cut, not the chest measurement. Two lines with the same `size` and `size_system` but different `fit` values are different garments.

## The `resolved_size` response field

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size.chest_cm` in order-confirmation emails and packing-list integrations instead of deriving measurements client-side.

## The `X-Printf-Size-System` response header

The response includes `X-Printf-Size-System` set to the system that was applied to the order. Useful for logging and debugging when the system was resolved from the account or facility default rather than from the payload.

## Fallback chain

If `size_system` is absent from both the line and the order, the API resolves it in this order:

1. Line `size_system`
2. Order `size_system`
3. Account default (set in account settings)
4. Fulfilling facility's default — **only if the account routes exclusively to one facility**

If the account can route to more than one facility and no `size_system` is present at line or order level, the request is rejected with **`400 size_system_ambiguous`**. This affects any account with multi-facility routing, including all accounts enrolled in the redundant-fulfilment programme.

## Warning and deprecation schedule

Single-facility accounts that omit `size_system` receive a `size_system_implicit` warning in the response body. This warning becomes a hard error in **order-api 2.6**. Add `size_system` now — do not wait for the error.

## Saved order templates

Templates do not appear in the API response and are easy to overlook. If you have saved templates, open each one in the dashboard or retrieve it with `GET /v2/order-templates/{templateId}` and confirm that `size_system` is set at the order or line level. A template submitted without `size_system` against a multi-facility account will fail at submission time with `400 size_system_ambiguous`.
