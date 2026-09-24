---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-09-24
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Bulk orders and templates

Saved order templates let you submit recurring orders without rebuilding the payload each time. As of 2.4.0, every template must carry an explicit `size_system`. Templates created before this release do not, and submitting them unmodified will return `400 size_system_ambiguous` for any account that routes to more than one facility.

:::danger
Audit your saved templates before deploying against 2.4.0. Any template missing `size_system` will fail at submission time for multi-facility accounts. Single-facility accounts will receive a `size_system_implicit` warning, which becomes an error in 2.6.
:::

## Updating existing templates

For each saved template:

1. Retrieve it via your template store or the API.
2. Add `size_system` at the order level.
3. Optionally add `size_system` and `fit` per line to be explicit about mixed-market orders.
4. Save the updated template.

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

## Bulk submission behaviour

When submitting a batch, each order in the batch is validated independently. A missing `size_system` on one order does not fail the entire batch — it returns `size_system_ambiguous` for that order only. Check per-order status in the response.

## Size system per line

For mixed-market bulk orders — for example, an event with US domestic and EU international attendees — set `size_system` per line rather than at the order level:

```json
{
  "accountId": "acct_stackfest",
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
      "quantity": 200,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "EU",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Line-level `size_system` takes precedence over the order-level value. See [Sizing and fit](/guides/sizing#resolution-precedence) for the full precedence table.

## Resolved size in responses

Every line in the response now includes `resolved_size`:

```json
{
  "resolved_size": {
    "label": "XL",
    "system": "US",
    "fit": "unisex",
    "chest_cm": 112
  }
}
```

Store this alongside your template records. It is the only way to confirm which ladder was applied to a fulfilled order.
