---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Bulk orders and templates

Breaking, as of Orders API 2.4.0: bulk orders and saved order templates that send bare `size` labels now fail or warn. Accounts that can route to more than one facility get `400 size_system_ambiguous`. Accounts that route to a single facility get a `size_system_implicit` warning, which becomes an error in 2.6.

## Size systems in a bulk order

Set `size_system` once on the order. Add it on a line only when that line uses a different ladder.

| Field | Where | Effect |
|---|---|---|
| `size_system` | Order | Default for every line that does not set its own. |
| `size_system` | Line | Overrides the order value for that line. |
| `fit` | Line | Set per line. Returned in `resolved_size`. |

The ladders differ, so check each line. A JP `XL` is 97 cm and a US `XL` is 112 cm.

`POST /v2/orders`

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
      "fit": "unisex",
      "quantity": 250,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "JP",
      "fit": "unisex",
      "quantity": 40,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

The first line inherits `US` from the order. The second line overrides it with `JP`. `accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

Each response line carries `resolved_size`. Check it on every line of a bulk order before you rely on the totals.

## Resolution order

| Order | Source |
|---|---|
| 1 | The line's `size_system` |
| 2 | The order's `size_system` |
| 3 | The account |
| 4 | The fulfilling facility's default |

Rule 4 depends on routing, not on your payload, so it is where multi-facility accounts are rejected with `size_system_ambiguous`.

## Saved order templates

Templates created before 2.4.0 carry no explicit `size_system`. Submitting one is the same as sending bare size labels.

| Your account | Submitting an old template |
|---|---|
| Can route to more than one facility | `400 size_system_ambiguous` |
| Routes to a single facility | `size_system_implicit` warning. Becomes an error in 2.6. |

Templates do not appear in API responses, so list them from wherever you manage them and from any automation that submits them. For each one:

1. Add `size_system` to the template, on the order or per line if it mixes ladders.
2. Submit one order from it and check `resolved_size` on each line.
3. Do the same for templates you rarely use.

Full before-and-after payload: [Migrating saved templates](/guides/sizing#migrating-saved-templates).

:::warning
Scheduled or automated jobs that submit a template will start failing without a code change on your side, because the template did not change but the API did.
:::

