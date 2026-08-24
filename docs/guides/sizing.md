---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, GET /v2/orders/{id}
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Sizing and fit

As of **2.4.0**, every size label must be accompanied by an explicit size system. Bare size labels are no longer silently resolved by guessing. See the [migration note](../guides/sizing.md#migration) if you are upgrading from 2.3.x.

## Size systems

| `size_system` value | Coverage |
|---|---|
| `US` | United States |
| `EU` | Europe (EN 13402) |
| `JP` | Japan (JIS L 4004) |

Set `size_system` at the order level to apply one system to every line. Override it per line when a single order mixes systems.

Ladders differ materially. Always check `resolved_size.chest_cm` in staging before sending production volume.

| Label | System | Chest (cm) |
|---|---|---|
| XL | US | 112 |
| XL | EU | 108 |
| XL | JP | 97 |

## Request structure

A complete order with explicit sizing:

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

## The `fit` field

Each line accepts an optional `fit` value. Supported values: `"unisex"`, `"womens"`, `"mens"`.

If omitted, the garment's default fit is used and reflected back in `resolved_size.fit`.

## The `resolved_size` response object

Every line in the response carries a `resolved_size` object:

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

| Field | Type | Description |
|---|---|---|
| `label` | string | Normalised size label as printed on the garment |
| `system` | string | The `size_system` that was applied (`US`, `EU`, or `JP`) |
| `fit` | string | Effective fit |
| `chest_cm` | number | Chest measurement in centimetres for this system and label |

The `X-Printf-Size-System` response header echoes the effective system for the whole order.

## Resolution order for bare labels

If `size_system` is absent on a line **and** on the order, the API resolves it with this fallback chain:

1. Line-level `size_system`
2. Order-level `size_system`
3. Account default
4. Fulfilling facility default

Step 4 depends on routing. If your account can route to more than one facility, the API cannot determine the facility before fulfilling, so it returns **`400 size_system_ambiguous`** instead of guessing. Single-facility accounts get a **`size_system_implicit`** warning today; that warning becomes a hard error in **2.6**.

**Recommendation:** always supply `size_system` explicitly. Do not rely on the fallback chain.

## Migration from 2.3.x {#migration}

See the full migration note in the [2.4.0 changelog](../changelog.md#2-4-0).
