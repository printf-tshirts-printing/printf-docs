---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform-integrations
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{orderId}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-rb
  - printf-go
---

# Sizing and fit

**Updated for order-api 2.4.0.** The API now requires an explicit size system. Bare size labels without a declared system are rejected for multi-facility accounts and warned for single-facility accounts. See the [migration note](#migration-from-bare-size-labels) below.

## Size systems

| `size_system` value | Coverage | Example: chest at `XL` |
|---|---|---|
| `US` | North America | 112 cm |
| `EU` | Europe | 106 cm |
| `JP` | Japan / East Asia | 97 cm |

Size ladders differ materially across systems. A US `XL` garment is 15 cm wider in the chest than a JP `XL`. Always declare the system your recipients expect.

## Declaring the size system

You can declare `size_system` at two levels. The more specific value wins.

| Level | Field | Scope |
|---|---|---|
| Order | `size_system` on the order body | Default for every line in the order |
| Line | `size_system` on an individual line | Overrides the order-level value for that line only |

If neither level is present, the API falls back to the account default, then the fulfilling facility's default — **but only for single-facility accounts**. Multi-facility accounts are rejected with `400 size_system_ambiguous` if no system is resolvable from the payload.

## The `fit` field

`fit` is declared per line. Accepted values are `unisex`, `mens`, and `womens`. When omitted, the garment's catalog default applies. Fit affects shoulder width and body taper; it does not change the size label.

## The `resolved_size` response object

Every line in a successful order response and in webhook payloads now includes a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

| Field | Type | Description |
|---|---|---|
| `label` | string | The size label as submitted |
| `system` | string | The `size_system` that was applied (`US`, `EU`, or `JP`) |
| `fit` | string | The fit that was applied |
| `chest_cm` | number | Resolved chest measurement in centimetres |

Use `resolved_size` in your order-confirmation flows to show recipients exactly what was produced.

## The `X-Printf-Size-System` response header

The response includes an `X-Printf-Size-System` header containing the size system that resolved the order. When `size_system` differs across lines, the header value is `mixed`.

```
X-Printf-Size-System: US
```

## Full example request and response

**Request**

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

**Response (line excerpt)**

```json
{
  "lines": [
    {
      "designId": "dsn_7fa91c",
      "size": "XL",
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

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | Multi-facility account submitted a bare size label with no resolvable `size_system`. Add `size_system` to the order or line. |
| `size_system_implicit` | — (warning) | Single-facility account submitted a bare size label. Request succeeded, but this becomes a hard error in **2.6**. |

## Migration from bare size labels

See the [migration note in the 2.4.0 changelog](../changelog/2.4.0.md#migration) for before/after diffs and saved-template instructions.

