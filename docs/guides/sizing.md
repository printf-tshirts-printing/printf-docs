---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
owner: product
covers_endpoints: ["POST /v2/orders"]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Sizing and fit

As of **order-api 2.4.0**, every order carries an explicit size system. Size labels alone are ambiguous — `XL` in US sizing is a 112 cm chest; `XL` in JP sizing is 97 cm. Printf needs to know which ladder you intend.

## Size systems

| `size_system` value | Region | Example: XL chest |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 104 cm |
| `JP` | Japan | 97 cm |

Set `size_system` at the **order level** as a default for all lines, then override per line when a single order mixes systems.

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
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

`fit` is set per line. Available values depend on the `garmentSku`. Typical values:

| `fit` | Description |
|---|---|
| `unisex` | Standard cut |
| `womens` | Contoured cut |
| `youth` | Youth proportions |

## Resolution chain

If you omit `size_system` on a line, the API resolves it in this order:

1. The line's own `size_system`
2. The order-level `size_system`
3. The account's configured default
4. The fulfilling facility's default

Step 4 depends on routing, not on your payload.

| Account type | Omitted `size_system` | Result |
|---|---|---|
| Single-facility | Any level | `size_system_implicit` warning (error in 2.6) |
| Multi-facility | Any level | `400 size_system_ambiguous` — request rejected |

> **Action required before 2.6:** Add `size_system` explicitly to every order and line. The `size_system_implicit` warning in the response body and the `X-Printf-Size-System` response header tell you which system was inferred — use that value to populate the field.

## `resolved_size` in responses

Every line in the order response and in webhook payloads includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `chest_cm` to verify the physical measurement matches your product spec before the order enters production. If `chest_cm` is not what you expected, the wrong size system was applied.

## Saved order templates

Saved templates are not visible through the API but they are rehydrated on every submission. **Templates created before 2.4.0 have no `size_system` field.** Each submission from a stale template will produce a `size_system_implicit` warning now and will be rejected in 2.6. Open each template in the dashboard, add `size_system`, and save.

