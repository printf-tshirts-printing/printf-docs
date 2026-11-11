---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py]
---

# Bulk orders and templates

Since order-api 2.4.0, a bulk order must say which size system its sizes use. This is a breaking change. A bare `size` on an account that can route to more than one facility fails with `400 size_system_ambiguous`.

Saved order templates are affected too. See [Saved templates](#saved-templates).

## Put many lines in one order

Set `size_system` once on the order. Every line without its own `size_system` uses it.

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
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 40,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The first line uses the order's `US` system, so its `XL` is 112 cm. The second line overrides it with `JP`, so its `XL` is 97 cm.

`accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

## Resolution order

A line with a bare `size` resolves its system from the line, then the order, then the account, then the fulfilling facility's default. The facility default depends on routing, not on your payload, so accounts that can route to more than one facility get `400 size_system_ambiguous` instead of a guess. See [Sizing and fit](/guides/sizing) for the full rules.

## Check every line

Each response line carries `resolved_size`. Webhook payloads carry it as well.

```json
{ "label": "XL", "system": "JP", "fit": "unisex", "chest_cm": 97 }
```

For a bulk order, compare `chest_cm` across lines before you rely on quantities.

## Saved templates

Saved order templates created before 2.4.0 carry no explicit `size_system`. Orders placed from them behave like bare sizes:

| Account | Result |
|---|---|
| Can route to more than one facility | `400 size_system_ambiguous` |
| Routes to a single facility | The order succeeds with a `size_system_implicit` warning. It becomes an error in 2.6. |

For each saved template:

1. Open the template and set `size_system` on the order.
2. If lines in the template use different systems, set `size_system` on those lines too.
3. Save the template.
4. Check `resolved_size` on the first order placed from it.

:::warning
On a single-facility account, an old template keeps working, using the facility default. A JP `XL` is 97 cm and a US `XL` is 112 cm, so a template written against the other ladder orders the wrong garments without an error. Check `resolved_size`.
:::

