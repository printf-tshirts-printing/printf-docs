---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks:
  - printf-js
  - printf-py
  - printf-java
  - printf-go
  - printf-rb
---

# Sizing and fit

As of order-api **2.4.0**, every size label must be paired with a size system. A bare `"XL"` means different things in different markets — a JP XL chest is 97 cm; a US XL is 112 cm. Printf no longer guesses.

## Size systems

| Value | Standard | Notes |
|---|---|---|
| `US` | US unisex / junior | Most North American fulfilment |
| `EU` | European numeric | Chest in cm varies by garment |
| `JP` | Japanese JIS | Smaller ladder than US or EU |

## Where to set `size_system`

You can set `size_system` at two levels. The per-line value wins when present; the order-level value fills in for any line that omits it.

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

The `size_system` on the line (`"US"`) overrides the order-level `"US"` here — both are included to show the pattern. If you set `size_system` once at the order level and leave it off each line, the order-level value applies to every line.

## `fit`

`fit` is a per-line field. `"unisex"` is accepted for all garments; garment-specific values are listed on the garment catalogue endpoint. Omitting `fit` does not cause an error today, but providing it improves size resolution accuracy.

## `resolved_size` in responses

Every response line and webhook payload now includes a `resolved_size` object:

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
| `label` | string | The normalised size label |
| `system` | string | The system used (`US`, `EU`, or `JP`) |
| `fit` | string | The fit used for resolution |
| `chest_cm` | number | Chest measurement in centimetres at this size |

Use `chest_cm` to validate that fulfilment will produce what you expect before the order ships.

## `X-Printf-Size-System` response header

The response header `X-Printf-Size-System` reflects the size system that was used to resolve sizes for the entire request. When lines use mixed systems this header contains the order-level system or `mixed` if no order-level system was provided.

## Resolution fallback chain

If `size_system` is omitted on a line, Printf resolves it in this order:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default

Step 4 depends on routing. If your account can route to **more than one facility**, Printf cannot determine a facility default without committing to a route, so the request is rejected with `400 size_system_ambiguous`. Set `size_system` explicitly to avoid this.

If your account is single-facility and you omit `size_system`, you receive a `size_system_implicit` warning today. This warning becomes an error (`400`) in **2.6**.

## Size ladders at a glance

| Size | US chest (cm) | JP chest (cm) |
|---|---|---|
| S | 96 | 83 |
| M | 104 | 90 |
| L | 112 | 97 — same label as US XL |
| XL | 112 | 97 |
| 2XL | 120 | 103 |

Review your `garmentSku` and `size` combinations against this table before migrating, especially if you source garments for international recipients.

