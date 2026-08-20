---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-20
owner: dx
covers_endpoints:
  - POST /v2/orders
covers_sdks:
  - printf-js
  - printf-py
  - printf-go
  - printf-java
  - printf-rb
---

# Sizing and fit

As of order-api **2.4.0**, bare size labels like `L` or `XL` are resolved against a named size ladder. You must tell Printf which ladder to use via `size_system`. Failing to do so produces a warning today (`size_system_implicit`) and a hard error in **2.6** (`size_system_ambiguous`).

> **SDK notice — size_system support pending.** `printf-js` (v2.3.4), `printf-py` (v2.3.2), `printf-go` (v2.3.1), `printf-java` (v3.2.1), and `printf-rb` (v2.3.0) all have `supportsSizeSystem: false`. Use raw HTTP calls for size-critical orders until updated SDK releases ship.

## Size systems

Three systems are supported:

| `size_system` | Coverage | Example facility |
|---|---|---|
| `US` | United States | Austin (`fac-atx`) |
| `EU` | Europe | Berlin (`fac-ber`) |
| `JP` | Japan | Osaka (`fac-osa`) |

## Size ladders

Ladders differ materially across systems. Always check chest measurements when switching systems.

### Chest measurement by system and size (cm)

| Label | US | EU | JP |
|---|---|---|---|
| S | 96 | 92 | 86 |
| M | 100 | 96 | 91 |
| L | 106 | 100 | 96 |
| XL | 112 | 106 | 97 |
| 2XL | 118 | 112 | 102 |

> A JP `XL` (97 cm) is equivalent to a US `M`/`L` boundary. Do not assume label equivalence across systems.

## The `fit` field

Each line accepts a `fit` value:

| Value | Description |
|---|---|
| `unisex` | Default cut |
| `mens` | Broader shoulders, longer body |
| `womens` | Tapered waist |
| `youth` | Children's proportions |

## Setting `size_system`

`size_system` can be set at the order root (applies to all lines) or overridden per line. The resolution order is: **line → order → account → facility default**.

```http
POST /v2/orders
Content-Type: application/json

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

The response includes `resolved_size` on every line:

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

The `X-Printf-Size-System` response header confirms the effective system used for the order.

## Error and warning codes

| Code | HTTP | When it fires | Becomes error in |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Account routes to multiple facilities and no `size_system` is set | Already an error |
| `size_system_implicit` | — (warning) | `size_system` resolved from facility default; single-facility accounts only | 2.6 |

## Saved order templates

Templates that omit `size_system` will receive `400 size_system_ambiguous` at runtime if the account routes to more than one facility. Add `size_system` to every template before running bulk jobs. See [Bulk orders and templates](docs/guides/bulk-orders.md).

