---
title: Sizing and fit
section: Guides
last_reviewed: 2026-08-21
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

As of order-api **2.4.0**, size labels are unambiguous. Every order resolves to an explicit size system — `US`, `EU`, or `JP` — and every response line carries a `resolved_size` object confirming exactly what was cut.

## Size systems

| System | Example: `XL` chest | Typical market |
|--------|---------------------|----------------|
| `US`   | 112 cm              | North America  |
| `EU`   | 104 cm              | Europe         |
| `JP`   | 97 cm               | Japan          |

A JP `XL` is 15 cm narrower than a US `XL`. Getting this wrong produces garments that arrive in the wrong size.

## Setting the size system

`size_system` can be placed at three levels; the most specific wins:

1. **Per line** — `lines[].size_system`
2. **Order level** — top-level `size_system`
3. **Account default** — set in the dashboard or via `PATCH /v2/accounts/{accountId}`

If none of those are present, the API falls back to the **fulfilling facility's default**. This fallback is only safe when your account routes to exactly one facility (see [Ambiguity and errors](#ambiguity-and-errors) below).

### Recommended request shape

Set `size_system` at order level and override per line only when an order contains mixed international garments.

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

`fit` is set per line. Accepted values depend on the garment SKU; consult the [catalog API](https://docs.printf.dev/api/catalog) for what each SKU supports. Common values: `unisex`, `fitted`, `relaxed`.

## The `resolved_size` response object

Every line in a successful order response and in webhook payloads now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

| Field | Type | Description |
|-------|------|-------------|
| `label` | string | The size label as submitted (`"XL"`, `"M"`, etc.) |
| `system` | string | The system that was applied (`US`, `EU`, or `JP`) |
| `fit` | string | The fit that was applied |
| `chest_cm` | integer | The chest measurement in cm for this garment |

You can use `chest_cm` for final QA checks before accepting a fulfillment confirmation.

## The `X-Printf-Size-System` response header

Every `POST /v2/orders` and `GET /v2/orders/{orderId}` response includes:

```
X-Printf-Size-System: US
```

This reflects the system applied at order level. When lines carry different systems, the header is omitted and `resolved_size.system` on each line is authoritative.

## Ambiguity and errors

### `400 size_system_ambiguous`

Returned when:
- No `size_system` is set at any level (line, order, or account), **and**
- Your account can route to more than one fulfillment facility with different size system defaults.

The API cannot guess which facility will win routing at fulfillment time. **Fix:** add `size_system` to the request.

### `size_system_implicit` warning

Returned (as a non-fatal warning alongside a `201`) when:
- No `size_system` is set at any level, **and**
- Your account routes to exactly one facility.

The order is accepted and the facility's default system is used, but the response body includes:

```json
{
  "warnings": [
    { "code": "size_system_implicit", "message": "Size system was not specified; resolved from facility default. This will be an error in 2.6." }
  ]
}
```

`size_system_implicit` **becomes a hard error (`400`) in 2.6.** Set `size_system` explicitly to avoid a breaking change at that upgrade.

## Saved order templates

Saved order templates are not visible in the API response but are evaluated at fulfillment time. If you use templates, open each one in the dashboard and add `size_system` explicitly. A template without `size_system` will produce `size_system_implicit` warnings (single-facility) or `400 size_system_ambiguous` errors (multi-facility) on every order it generates after 2.4.0.

