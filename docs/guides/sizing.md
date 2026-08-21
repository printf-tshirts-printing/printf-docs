---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: ["POST /v2/orders"]
covers_sdks: ["printf-js","printf-py","printf-java","printf-go","printf-rb"]
---

# Sizing and fit

As of order-api **2.4.0**, size resolution is explicit. A `size` label like `"L"` means different things in different markets — a JP `XL` has a 97 cm chest; a US `XL` has a 112 cm chest. Printf no longer guesses.

## Size system

Set `size_system` once on the order to apply it to every line, or override it per line.

Accepted values: `US`, `EU`, `JP`.

**Order-level (applies to all lines)**

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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

**Per-line override**

Add `size_system` directly on the line to override the order-level value for that line only.

```json
{
  "designId": "dsn_7fa91c",
  "garmentSku": "tee-classic-black",
  "size": "XL",
  "size_system": "JP",
  "fit": "unisex",
  "quantity": 50
}
```

## Fit

`fit` is accepted per line. Valid values: `unisex`, `womens`, `mens`. When omitted the fulfilling facility's default fit for the garment SKU is used.

## Fallback resolution chain

When `size_system` is absent on a line, Printf resolves in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility's default

Step 4 depends on routing. If your account can route to **more than one facility** and you omit `size_system`, the request is rejected with `400 size_system_ambiguous`. If your account routes to a single facility only, the facility default is used and a `size_system_implicit` warning is attached to the response. **That warning becomes an error in 2.6.**

## `resolved_size` in responses

Every line in a successful response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header carries the system that resolved at the order level.

## Reference table

| Size label | US chest (cm) | EU chest (cm) | JP chest (cm) |
|---|---|---|---|
| S | 91 | 88 | 84 |
| M | 99 | 96 | 91 |
| L | 107 | 104 | 97 |
| XL | 112 | 109 | 97 |
| 2XL | 122 | 119 | 107 |

## Saved order templates

Templates are not updated automatically. If you have saved templates that omit `size_system`, they will trigger `size_system_implicit` warnings now and fail with an error in 2.6. Open each template and add `size_system` at the order level or per line.

