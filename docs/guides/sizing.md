---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-21
owner: fulfillment
covers_endpoints:
  - POST /v2/orders
  - GET /v2/orders/{id}
covers_sdks: []
---

# Sizing and fit

As of order-api 2.4.0, every order and line carries an explicit size system. This page explains the fields, the resolution ladder, and what happens when `size_system` is absent.

## Size system field

`size_system` accepts three values:

| Value | Region | Example: XL chest |
|---|---|---|
| `US` | United States / Canada | 112 cm |
| `EU` | Europe | 106 cm |
| `JP` | Japan | 97 cm |

A JP `XL` and a US `XL` differ by 15 cm. Specify the wrong system and the garment arrives in the wrong size. This cannot be corrected after production.

## Where to set `size_system`

`size_system` can appear at two levels:

| Level | Field location | Scope |
|---|---|---|
| Order | Top-level request body | Applies to all lines that do not set their own `size_system` |
| Line | Inside each `lines[]` entry | Overrides the order-level value for that line only |

Set it on every line when your order mixes systems (rare). Set it once at order level when the whole order is one system (typical).

```json
{
  "accountId": "acct_stackfest",
  "size_system": "US",
  "destination": { "name": "StackFest Ops", "line1": "410 Congress Ave", "city": "Austin", "region": "TX", "postalCode": "78701", "countryCode": "US" },
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

## Fit

The `fit` field is per-line. Accepted values depend on the garment SKU — consult your account catalog. Common values: `unisex`, `mens`, `womens`, `youth`.

## Resolution ladder

When `size_system` is omitted, the API resolves it in this order:

1. **Line** — `lines[].size_system`
2. **Order** — top-level `size_system`
3. **Account default** — configured on your Printf account
4. **Fulfilling facility default** — determined by routing

Step 4 depends on routing, not on your payload.

### Multi-facility accounts

If your account can route to more than one facility, and no `size_system` is set at line, order, or account level, the API cannot safely pick a facility default. The request is rejected:

```
400 size_system_ambiguous
```

The only fix is to add `size_system` explicitly. See the [migration note](../guides/bulk-orders.md#migration-240) for before/after examples.

### Single-facility accounts

The facility default is unambiguous, so the request is accepted. The response includes a `size_system_implicit` warning. **This warning becomes a `400` error in order-api 2.6.** Add `size_system` now.

## `resolved_size` response field

Every response line includes `resolved_size`:

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

Verify `chest_cm` before fulfillment. The `X-Printf-Size-System` response header contains the system applied to the order as a whole.

## Ladder reference

### US

| Label | Chest (cm) |
|---|---|
| S | 91 |
| M | 99 |
| L | 107 |
| XL | 112 |
| 2XL | 117 |

### JP

| Label | Chest (cm) |
|---|---|
| S | 82 |
| M | 88 |
| L | 94 |
| XL | 97 |
| 2XL | 102 |

### EU

| Label | Chest (cm) |
|---|---|
| S | 88 |
| M | 96 |
| L | 102 |
| XL | 106 |
| 2XL | 112 |

