---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

As of order-api **2.4.0**, every size label must be paired with a size system. A bare `size` without a `size_system` is either rejected or warned, depending on your account's routing configuration. See the [migration note](#migration-note-from-pre-240) below.

## Size systems

| Value | Region | US `XL` equivalent chest |
|---|---|---|
| `US` | North America | 112 cm |
| `EU` | Europe | 107 cm |
| `JP` | Japan / Asia-Pacific | 97 cm |

Size ladders differ materially. Sending the wrong system produces wrong-size garments with no post-fulfillment recourse.

## Where to set `size_system`

`size_system` can be set at three places and resolves in this order, most-specific first:

1. **Per line** — `lines[].size_system`
2. **Order root** — top-level `size_system`
3. **Account default** — configured on your account in the dashboard

If none of these is set and your account routes to a single facility, the facility default is used but the response includes a `size_system_implicit` warning. If your account routes to more than one facility, the request is rejected with `400 size_system_ambiguous`.

## `fit`

Set `fit` per line. Accepted values depend on the garment SKU; common values are `unisex`, `womens`, and `mens`. Omitting `fit` uses the garment's catalog default.

## `resolved_size` in responses

Every response line now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The same object appears on webhook payloads. Use `chest_cm` for downstream display and auditing — it is the canonical measurement that drove fulfillment.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` contains the size system applied to the order as a whole (e.g. `US`). Log this alongside your order ID for debugging.

## Full request example

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

Response excerpt:

```json
{
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "quantity": 250,
      "resolved_size": {
        "label": "XL",
        "system": "US",
        "fit": "unisex",
        "chest_cm": 112
      }
    }
  ]
}
```

## Migration note from pre-2.4.0

See the [migration note in the changelog](../changelog.md#order-api-240--size-disambiguation) for the full before/after diff, affected accounts, and the timeline for `size_system_implicit` becoming a hard error in 2.6.

