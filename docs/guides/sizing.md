---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
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

As of **2.4.0**, every size label must be accompanied by an explicit size system. Silent fallback to a facility default is removed for multi-facility accounts and deprecated for single-facility accounts.

## Size systems

| `size_system` value | Region | Example: chest at `XL` |
|---|---|---|
| `US` | United States | 112 cm |
| `EU` | Europe | 104 cm |
| `JP` | Japan | 97 cm |

Ladders differ materially. A JP `XL` is 15 cm narrower in the chest than a US `XL`. Always specify the system that matches your size label source.

## Where to set `size_system`

`size_system` can be set at two levels. The line value takes precedence over the order value.

```
Order.size_system          ← fallback for all lines that omit it
  Line.size_system         ← overrides the order value for that line
```

Set it at the order level when all lines share one system. Override per line when a single order mixes systems (for example, US-sized tees and JP-sized hoodies in one bulk run).

## `fit`

`fit` is set per line. Accepted values: `unisex`, `mens`, `womens`. This field determines which size ladder Printf uses for garments that ship separate cuts per gender.

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
      "garmentSku": "tee-classic-black",
      "size": "XL",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 250
    }
  ]
}
```

## `resolved_size` in responses

Every line in the order response and in webhook payloads now includes a `resolved_size` object:

```json
"resolved_size": {
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

Use `resolved_size` to confirm that the system Printf applied matches your expectation before the order enters production.

## `X-Printf-Size-System` response header

Every order response includes an `X-Printf-Size-System` header echoing the resolved system. Useful for logging and for asserting system consistency in integration tests without parsing the body.

```
X-Printf-Size-System: US
```

## Errors and warnings

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | `400` | No `size_system` on request and account routes to more than one facility. Printf cannot pick a system safely. | Add `size_system` to the order or the affected line. |
| `size_system_implicit` | — (warning) | No `size_system` on request and account routes to exactly one facility. Request succeeded, but the fallback is deprecated. | Add `size_system` before 2.6, when this becomes a `400`. |

## Saved order templates

Templates are not validated at save time. If you have saved templates that omit `size_system`, they will produce `size_system_ambiguous` or `size_system_implicit` at submission time depending on your account's facility routing. **Audit your templates and add `size_system` to each one.**

