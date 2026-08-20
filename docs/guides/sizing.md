---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-20
owner: docs-platform
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

As of order-api **2.4.0**, bare size labels like `"XL"` are resolved against a named size system. Ladders differ materially across systems — a JP `XL` has a 97 cm chest; a US `XL` has a 112 cm chest. Getting this wrong ships the wrong garment.

> **SDK notice:** `printf-js`, `printf-py`, `printf-go`, `printf-java`, and `printf-rb` all report `supportsSizeSystem=false`. They do not yet model `size_system`, `fit`, or `resolved_size`. Use raw HTTP calls or wait for updated SDK releases if you need these fields through a client library.

## Size systems

| Code | Description | Facilities that default to this system |
|---|---|---|
| `US` | US standard | `fac-atx` (Austin) |
| `EU` | European standard | `fac-ber` (Berlin) |
| `JP` | Japanese standard | `fac-osa` (Osaka) |

## Chest measurement ladders

### US

| Size label | Chest (cm) |
|---|---|
| S | 91 |
| M | 99 |
| L | 107 |
| XL | 112 |
| 2XL | 117 |

### EU

| Size label | Chest (cm) |
|---|---|
| S | 88 |
| M | 96 |
| L | 104 |
| XL | 110 |
| 2XL | 116 |

### JP

| Size label | Chest (cm) |
|---|---|
| S | 82 |
| M | 89 |
| L | 96 |
| XL | 97 |
| 2XL | 104 |

> These tables are informational. The authoritative value for any order line is `resolved_size.chest_cm` on the API response.

## Setting size_system on a request

Set `size_system` at the line level for precision. Set it at the order level as a default for all lines. A line-level value always overrides the order-level value.

```json
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

## fit

The `fit` field is set per line. It contributes to size resolution and is echoed back in `resolved_size.fit`.

Accepted values depend on the garment SKU. `"unisex"` is accepted on all SKUs.

## Resolution chain

When `size_system` is absent on a line, the API walks this chain and uses the first value it finds:

1. `size_system` on the line
2. `size_system` on the order
3. Default configured on the account
4. Default of the fulfilling facility

**Multi-facility accounts:** if resolution reaches step 4 and your account can route to more than one facility, the request fails with `400 size_system_ambiguous`. The facility default is routing-dependent and cannot be determined from the payload alone. Set `size_system` explicitly.

**Single-facility accounts:** resolution to step 4 succeeds today but emits a `size_system_implicit` deprecation warning in the response. This becomes a hard `400` error in **2.6.0**.

## Verifying resolved size

Every line in the response carries `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

The `X-Printf-Size-System` response header also reports the system applied to the order.

Always check `resolved_size.chest_cm` in your test environment when switching facilities or size systems — the same label can differ by 15 cm across systems.

