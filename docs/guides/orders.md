---
title: Orders
section: guides
last_reviewed: 2026-08-21
owner: devex
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-go
  - printf-java
  - printf-rb
---

# Orders

## Creating an order

`POST /v2/orders`

### Required fields

| Field | Type | Description |
|---|---|---|
| `accountId` | string | Your account identifier |
| `destination` | object | Shipping destination (see below) |
| `lines` | array | One or more order lines |

Each line requires `designId`, `garmentSku`, `quantity`, and `size`.

### Sizing fields (added in 2.4.0)

As of 2.4.0, you should include `size_system` on the order and on each line. See [Sizing and fit](/guides/sizing-and-fit) for the full field reference and resolution rules.

| Field | Scope | Type | Description |
|---|---|---|---|
| `size_system` | Order | `"US"` \| `"EU"` \| `"JP"` | Default size ladder for all lines |
| `size_system` | Line | `"US"` \| `"EU"` \| `"JP"` | Overrides order-level `size_system` for this line |
| `fit` | Line | `"unisex"` \| `"womens"` \| `"mens"` | Defaults to `"unisex"` |

Accounts routing to more than one facility **must** send `size_system`. Omitting it returns `400 size_system_ambiguous`.

### Example request

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

### Response

Lines in the response include a `resolved_size` object:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header echoes the resolved size system for the order.

## Error codes

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | `size_system` omitted and account routes to multiple facilities |
| `size_system_implicit` | warning | `size_system` resolved from facility default; becomes an error in 2.6 |

For the full error list see [Errors](/guides/errors).

