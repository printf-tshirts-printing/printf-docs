---
title: Create an order
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints:
  - POST /v2/orders
covers_sdks: []
---

# Create an order

This guide walks you through placing a production order against the Printf API.

## Minimal request

The fields below are **required on every request**. Omitting any of them returns `400`.

| Field | Type | Description |
|---|---|---|
| `accountId` | string | Your Printf account identifier |
| `destination` | object | Shipping address (see [Destination fields](#destination-fields)) |
| `lines[].designId` | string | Design to print |
| `lines[].garmentSku` | string | Blank garment SKU |
| `lines[].quantity` | integer | Units to produce |

## Size system (added in 2.4.0)

Size ladders differ materially between regions. A JP `XL` chest is 97 cm; a US `XL` chest is 112 cm. Sending the wrong size system ships the wrong garment.

Specify `size_system` at the order level, at the line level, or both. Line-level takes precedence.

| Value | Region |
|---|---|
| `US` | United States |
| `EU` | Europe |
| `JP` | Japan |

**If you omit `size_system`** Printf walks this resolution chain:

1. Line-level `size_system`
2. Order-level `size_system`
3. Your account default
4. The fulfilling facility's default

Step 4 is only reachable when your account is locked to a single facility. If your account can route to more than one facility and no `size_system` is found before step 4, the request is rejected with `400 size_system_ambiguous`. See the [migration note](../guides/size-system-migration.md) for how to fix this.

Even when step 4 succeeds, the API returns a `size_system_implicit` warning. **This warning becomes an error in 2.6.** Set an explicit `size_system` now.

## Example request

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

## Example response (lines)

Each line in the response now includes `resolved_size`:

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

The response also carries the header:

```
X-Printf-Size-System: US
```

## Destination fields

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | ✅ | Recipient name or company |
| `line1` | string | ✅ | Street address |
| `city` | string | ✅ | City |
| `region` | string | ✅ | State / province code |
| `postalCode` | string | ✅ | Postal / ZIP code |
| `countryCode` | string | ✅ | ISO 3166-1 alpha-2 country code |

