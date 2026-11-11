---
title: Sizing and fit
section: guides
last_reviewed: 2026-11-11
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/sizes]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

Since order-api 2.4.0, `size` is no longer interpreted without a size system. This is a breaking change. If your account can route to more than one facility, a request with a bare size such as `XL` fails with `400 size_system_ambiguous`.

This page covers how sizes resolve, the codes you can hit, and how to migrate saved order templates.

## Why sizes need a system

Size ladders differ materially between systems.

| Label | System | Chest |
|---|---|---|
| `XL` | `JP` | 97 cm |
| `XL` | `US` | 112 cm |

A bare `XL` could therefore produce garments 15 cm apart in chest width.

## Send `size_system` and `fit`

`size_system` accepts `US`, `EU` or `JP`. Set it on the order, on a line, or both. `fit` is set per line.

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

`accountId`, `destination`, `designId`, `garmentSku` and `quantity` are required on every request.

## How a bare size resolves

If a line has a `size` but no system, Printf looks for one in this order and uses the first it finds.

| Order | Source |
|---|---|
| 1 | `size_system` on the line |
| 2 | `size_system` on the order |
| 3 | The account |
| 4 | The fulfilling facility's default |

Step 4 depends on routing, not on your payload. If resolution reaches step 4 and your account can route to more than one facility, Printf rejects the request with `400 size_system_ambiguous` rather than guessing. Single-facility accounts resolve at step 4 and get a `size_system_implicit` warning.

## Codes

| Code | Kind | When | Action |
|---|---|---|---|
| `size_system_ambiguous` | Error, `400` | A bare size reaches the facility default and your account can route to more than one facility | Send `size_system` on the order or the line |
| `size_system_implicit` | Warning | A bare size resolved from the facility default on a single-facility account | Send `size_system`. This becomes an error in 2.6. |

## Read what was resolved

Each response line carries `resolved_size`. Webhook payloads carry it too.

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

Responses also carry an `X-Printf-Size-System` header.

## Migrating saved templates

Saved order templates created before 2.4.0 carry no explicit `size_system`. They behave like bare sizes: multi-facility accounts get `400 size_system_ambiguous`, and single-facility accounts get `size_system_implicit`.

Templates are invisible from the API, so check them in your account. For each saved template:

1. Open the template and set `size_system` on the order.
2. If lines in the template use different systems, set `size_system` on those lines too.
3. Save the template.
4. Check `resolved_size` on the first order placed from it.

:::warning
Single-facility accounts do not get an error today for a bare size. The facility default is applied and only a `size_system_implicit` warning is returned. A template that was written against a different ladder can order the wrong garments, for example a JP `XL` of 97 cm instead of a US `XL` of 112 cm. Check `resolved_size` on template orders.
:::

## Deadline

`size_system_implicit` becomes an error in 2.6. Multi-facility accounts already get `size_system_ambiguous` in 2.4.0.

