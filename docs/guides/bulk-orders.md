---
title: Bulk orders and templates
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
  - printf-rb
  - printf-go
---

# Bulk orders and templates

This guide covers submitting large orders and managing reusable order templates. As of **order-api 2.4.0**, every template and every bulk payload must carry an explicit `size_system`. Bare size labels in a bulk request are rejected with `400 size_system_ambiguous` if your account can route to more than one facility.

## Bulk order payload

A bulk order is a single `POST /v2/orders` request with many lines. There is no separate bulk endpoint. The same required fields apply regardless of line count.

**Required fields on every request:** `accountId`, `destination`, and for each line: `designId`, `garmentSku`, `quantity`.

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
      "size": "S",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 150
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 300
    },
    {
      "designId": "dsn_7fa91c",
      "garmentSku": "tee-classic-black",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 300
    },
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

Set `size_system` at the order root as a default and optionally override it per line. The per-line value always wins.

## Response lines and `resolved_size`

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

In a bulk context this is your authoritative record of what was cut. Log or store it — do not re-derive sizes from labels alone.

## Templates

Order templates let you save a partial order payload and submit it repeatedly with overrides. As of 2.4.0 templates are subject to the same `size_system` rules as live requests.

### What changes for existing templates

Templates are not visible in API list responses, but they are validated fully at submission time. A template created before 2.4.0 that stores bare size labels will behave as follows when submitted:

| Account routing | Behaviour | Error / warning |
|---|---|---|
| Can reach multiple facilities | Rejected immediately | `400 size_system_ambiguous` |
| Single-facility only | Accepted with warning | `size_system_implicit` warning — becomes `400` in **2.6.0** |

**Action required:** before 2.6.0, open every template and add `size_system` to the order root and to every line that carries a `size` value.

### Template submission with size system

When you submit a template, include `size_system` in the override payload if the template itself does not already carry it:

```json
POST /v2/orders
{
  "accountId": "acct_stackfest",
  "templateId": "tmpl_annual_tee",
  "size_system": "US",
  "destination": {
    "name": "StackFest Ops",
    "line1": "410 Congress Ave",
    "city": "Austin",
    "region": "TX",
    "postalCode": "78701",
    "countryCode": "US"
  }
}
```

The override `size_system` is applied to every line in the template that does not carry its own value.

## Mixed-system bulk orders

If you are ordering for audiences in multiple regions in a single request, set `size_system` per line:

```json
"lines": [
  {
    "designId": "dsn_7fa91c",
    "garmentSku": "tee-classic-black",
    "size": "XL",
    "size_system": "US",
    "fit": "unisex",
    "quantity": 250
  },
  {
    "designId": "dsn_7fa91c",
    "garmentSku": "tee-classic-black",
    "size": "XL",
    "size_system": "EU",
    "fit": "unisex",
    "quantity": 100
  }
]
```

The `X-Printf-Size-System` response header will read `mixed` when lines carry more than one system and no single order-level default covers them all.

## Error reference

| Code | HTTP status | Meaning | Fix |
|---|---|---|---|
| `size_system_ambiguous` | 400 | Bare size in a multi-facility account | Add `size_system` to every line or the order root |
| `size_system_implicit` | — (warning) | Bare size resolved by single-facility default | Add `size_system` before 2.6.0 |

