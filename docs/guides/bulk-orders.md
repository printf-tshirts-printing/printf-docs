---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-28
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Bulk orders and templates

This page covers creating large orders with many lines and reusing order configurations with saved templates. If you are new to the Orders API, read [Quickstart](/guides/quickstart) first.

:::danger
Orders API 2.4.0 requires an explicit `size_system` on every order. Saved templates created before 2.4.0 carry none. Templates used by accounts that route to more than one facility will fail immediately with `400 size_system_ambiguous`. All templates without `size_system` will fail in **2.6**. See [updating existing templates](#updating-existing-templates).
:::

## Bulk orders

A single `POST /v2/orders` call can contain up to 500 lines. Each line must have `designId`, `size`, `quantity`, and `garmentSku`. As of 2.4.0, `size_system` is also required unless your account has a single facility and you are willing to accept a `size_system_implicit` warning (deprecated; removed in 2.6).

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
      "size": "S",
      "size_system": "US",
      "fit": "fitted",
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "fitted",
      "quantity": 160,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "fitted",
      "quantity": 160,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
      "size_system": "US",
      "fit": "fitted",
      "quantity": 100,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The response includes `resolved_size` on each line so you can verify the system that was applied:

```json
{
  "lines": [
    {
      "resolved_size": { "label": "XL", "system": "US", "fit": "fitted", "chest_cm": 112 }
    }
  ]
}
```

## Saved templates

A saved order template stores an order payload that can be reused without re-submitting all fields. Templates are useful for recurring events where the destination and line composition stay the same across runs.

### Creating a template (2.4.0+)

Always include `size_system`. A template without it will warn today and fail in 2.6.

```json
POST /v2/order-templates
{
  "accountId": "acct_stackfest",
  "name": "StackFest annual tee run",
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

### Updating existing templates {#updating-existing-templates}

Templates saved before 2.4.0 carry no `size_system`. Follow these steps:

1. Fetch all templates: `GET /v2/order-templates?accountId={accountId}`
2. For each template, check whether `size_system` is present at the order level or on every line.
3. Add `size_system` at the order level if all lines share a system, or per line if they differ.
4. `PUT /v2/order-templates/{templateId}` with the updated payload.

```diff
  {
    "accountId": "acct_stackfest",
    "name": "StackFest annual tee run",
+   "size_system": "US",
    "lines": [
      {
        "designId": "dsn_7fa91c",
        "size": "XL",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

**Deadline:** accounts routing to multiple facilities break immediately. All other accounts break in **2.6** when `size_system_implicit` becomes an error.

## Line-level versus order-level `size_system`

Line-level always takes precedence. Set `size_system` at the order level as a default, then override per line where your run includes garments for different markets.

| Field location | Applies to |
|----------------|------------|
| `size_system` on the order | All lines that do not set their own |
| `size_system` on a line | That line only |

## Webhook payloads

Webhook payloads for bulk orders include `resolved_size` on each line, matching the response structure. Use this to confirm which system was applied before downstream fulfilment processing.

