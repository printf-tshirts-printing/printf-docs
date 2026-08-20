---
title: Orders
section: api
generated_from: order-api/openapi.yaml
last_reviewed: 2026-07-28
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/orders/{id}]
---

# Orders

<!-- GENERATED FROM order-api/openapi.yaml — DO NOT EDIT BY HAND -->

## Create an order

`POST /v2/orders`

### Request

| Field | Type | Required | Description |
|---|---|---|---|
| `accountId` | string | yes | Your Printf account. |
| `lines` | array | yes | At least one order line. |
| `destination` | object | yes | Where the finished order ships. |
| `facilityId` | string | no | Pin to `fac-atx`, `fac-ber` or `fac-osa`. Omit to let routing decide. |

#### Order line

| Field | Type | Required | Description |
|---|---|---|---|
| `designId` | string | yes | A design that has passed art validation. |
| `size` | string | yes | One of `XS` `S` `M` `L` `XL` `2XL` `3XL`. |
| `quantity` | integer | yes | 1–5000. |
| `garmentSku` | string | yes | Blank garment, e.g. `tee-classic-black`. |

### Response `201`

```json
{
  "id": "ord_9f2c4a7b1e6d05",
  "accountId": "acct_stackfest",
  "status": "accepted",
  "facilityId": "fac-atx",
  "lines": [{ "designId": "dsn_7fa91c", "size": "L", "quantity": 250, "garmentSku": "tee-classic-black" }],
  "createdAt": "2026-08-03T09:12:44Z",
  "warnings": []
}
```

### Errors

| Code | Status | Meaning |
|---|---|---|
| `size_unavailable` | 400 | The size label is not one we print. |
| `art_too_large` | 400 | Design file exceeds 40 MB. |
| `routing_unavailable` | 503 | Could not reach fulfillment routing. |
