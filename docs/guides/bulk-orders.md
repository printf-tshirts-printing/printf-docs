---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Bulk orders and templates

:::danger
Saved order templates created before Orders API 2.4.0 carry no `size_system`. They will produce a `size_system_implicit` warning on submission today and a `400` error from 2.6 onward. Update every template before upgrading, or before 2.6 ships — whichever comes first.
:::

## Sending a bulk order

Bulk orders follow the same `POST /v2/orders` shape as single orders. Each line is an independent entry in the `lines` array. As of 2.4.0, `size_system` is required on the order or on every line.

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
      "size": "S",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 50,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 120,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 80,
      "garmentSku": "tee-classic-black"
    }
  ]
}
```

Each line in the response will carry `resolved_size` confirming the system and chest measurement applied:

```json
{
  "label": "L",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 107
}
```

## Mixed size systems in a single order

You can mix size systems across lines by omitting the order-level `size_system` and specifying it per line. Every line must then carry its own `size_system`; an order with some lines missing it and no order-level default will be rejected with `400 size_system_ambiguous` on multi-facility accounts.

## Saved order templates

### What templates stored before 2.4.0

Templates saved before 2.4.0 contain a size label (`S`, `M`, `L`, `XL`, `XXL`) but no `size_system`. When you submit one today, the API applies the `size_system_implicit` warning path — the facility default is used if the account routes to a single facility. This becomes a `400` in 2.6.

### Updating your templates

There is no bulk migration endpoint. For each saved template:

1. Retrieve the template.
2. Add `size_system` at the order level — use whichever system your garments are sized in.
3. Optionally add `size_system` and `fit` per line for finer control.
4. Save the template back.

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
    "lines": [
      {
        "designId": "dsn_7fa91c",
        "size": "XL",
+       "size_system": "US",
+       "fit": "unisex",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

### Verifying the update

After saving the updated template, submit a test order (or a dry-run if your account has that capability enabled) and confirm that:

- The response does **not** contain a `size_system_implicit` warning.
- Each line's `resolved_size.system` matches the system you specified.
- `resolved_size.chest_cm` matches the ladder for that system (US `XL` = 112 cm, JP `XL` = 97 cm).

## Error reference

| Code | HTTP status | Meaning |
|---|---|---|
| `size_system_ambiguous` | 400 | No `size_system` resolvable; account routes to more than one facility. |
| `size_system_implicit` | Warning (200) | No `size_system`; single-facility account. Becomes 400 in 2.6. |
