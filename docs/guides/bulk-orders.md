---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Bulk orders and templates

Breaking change in order-api 2.4.0: bulk orders and saved templates that send a bare `size` stop working on multi-facility accounts. They are rejected with `400 size_system_ambiguous`. On single-facility accounts they return a `size_system_implicit` warning, which becomes an error in 2.6.

## Who needs to act

| Your setup | Effect in 2.4.0 | Action |
|---|---|---|
| Account can route to more than one facility | `400 size_system_ambiguous` | Send `size_system` on every order. |
| Account routes to a single facility | `size_system_implicit` warning | Send `size_system` before 2.6. |
| Saved templates created before 2.4.0 | No explicit `size_system` | Update each template. See [Migrating saved templates](/guides/sizing#migrating-saved-templates). |

## Mixing size systems in one order

Set `size_system` on the order for the common case. Override it on a line that differs. A line value wins over the order value.

The example below orders 250 US `XL` and 40 JP `XL`. These are different garments: a US `XL` is 112 cm chest and a JP `XL` is 97 cm.

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

The first line has no `size_system`, so it takes the order's `US`. The second line overrides it with `JP`.

## Checking what you ordered

Every response line, and every webhook payload for the order, carries `resolved_size`:

| Field | Example |
|---|---|
| `label` | `XL` |
| `system` | `US` |
| `fit` | `unisex` |
| `chest_cm` | `112` |

For a bulk order, compare `chest_cm` on each line with what you meant before the order is fulfilled. The response also carries an `X-Printf-Size-System` header.

## Saved templates

Templates created before 2.4.0 carry no explicit `size_system`.

:::warning
A template without `size_system` either fails (multi-facility accounts) or quietly resolves to the fulfilling facility's default (single-facility accounts). That default may not be the system your team means.
:::

For each template, set `size_system` on the order and on any line that differs, then place a test order and check `resolved_size`. The full steps and a before/after diff are in [Migrating saved templates](/guides/sizing#migrating-saved-templates).

