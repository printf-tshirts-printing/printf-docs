---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform-docs
covers_endpoints: POST /v2/orders, GET /v2/orders/{orderId}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

As of order-api **2.4.0**, every size label must be accompanied by a size system. Bare labels without a system are ambiguous because ladders differ materially across regions — a JP `XL` is 97 cm chest, a US `XL` is 112 cm.

## Size systems

| Value | Ladder | Notes |
|---|---|---|
| `US` | US/CA standard | Default for North American facilities |
| `EU` | European standard | Used for EU and UK fulfillment |
| `JP` | Japanese standard | Materially smaller than US at the same label |

## Where to set `size_system`

You can declare the system at three levels. Resolution works line → order → account.

| Level | Field | Scope |
|---|---|---|
| Line | `lines[].size_system` | Overrides everything for that line |
| Order | `size_system` | Applies to all lines that omit their own |
| Account | Account default (configured in the dashboard) | Fallback when neither order nor line sets a system |

If none of those are set and your account can route to **more than one facility**, the request is rejected with `400 size_system_ambiguous`. There is no silent guess. Single-facility accounts receive a `size_system_implicit` warning in the response until 2.6, at which point that also becomes an error.

## Setting fit

Each line accepts a `fit` value: `unisex`, `mens`, or `womens`. Fit affects the cut of the garment, not the size ladder. If omitted, the fulfilling facility's default fit applies.

## Full example

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

## Response: `resolved_size`

Every line in the response now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The same object appears in webhook payloads for order confirmation and shipment events.

## Response header

`X-Printf-Size-System` is set on every order response. Its value is the system that was applied when resolving sizes — useful for logging and debugging cross-region orders.

## Warnings and errors

| Code | HTTP status | Meaning | Becomes error in |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and no system was declared | Already an error |
| `size_system_implicit` | 200 (warning) | Single-facility account; system was inferred from facility default | 2.6 |

Check the `warnings` array in every 200 response. A `size_system_implicit` entry means you have implicit reliance on a facility default that will break in 2.6.

## Saved order templates

Templates do not automatically inherit the new fields. If you have saved templates, open each one and add `size_system` and `fit` before 2.6 ships. Templates are not visible from the orders API — edit them from the dashboard or your template management tooling.

