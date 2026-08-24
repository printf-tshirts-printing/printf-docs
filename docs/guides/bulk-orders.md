---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-08-24
owner: platform
covers_endpoints: POST /v2/orders, POST /v2/orders/batch
covers_sdks: printf-js, printf-py, printf-java, printf-go, printf-rb
---

# Bulk orders and templates

This guide covers high-volume order submission and the saved-template workflow. If you use templates, read the [size system notice](#size-system-and-templates) before your next submission.

## Submitting a bulk order

For orders with many lines, include all lines in a single `POST /v2/orders` request. The API processes lines atomically — the whole order succeeds or the whole order fails.

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
      "quantity": 300,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "M",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 500,
      "garmentSku": "tee-classic-black"
    },
    {
      "designId": "dsn_7fa91c",
      "size": "L",
      "size_system": "US",
      "fit": "unisex",
      "quantity": 500,
      "garmentSku": "tee-classic-black"
    },
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

Set `size_system` at the order level when every line uses the same system. Override it per line only when a single order genuinely mixes markets.

## Verifying resolved sizes before production

The response includes a `resolved_size` object on each line:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

For bulk orders, iterate the response lines and assert that every `chest_cm` matches your inventory data **before** the order enters the production queue. A mismatch is far cheaper to catch here than after cut.

## Size system and templates

> **Action required if you use saved order templates.**

Templates store the payload you submitted when you saved them. Any template created before **2.4.0** does not include `size_system`. When you submit that template:

- **Single-facility accounts** — the request goes through using your facility's default system, but you receive a `size_system_implicit` warning. This warning becomes an error in **2.6**.
- **Multi-facility accounts** — the request is immediately rejected with `400 size_system_ambiguous`.

**To fix:** open each template, add `size_system` at the order level (and per line if a template mixes markets), and save.

## Splitting across facilities

If you route lines to different facilities — for example, domestic and international fulfillment — supply `size_system` per line. Different facilities may have different default size ladders, and the order-level `size_system` only covers lines that do not specify their own.

## Rate limits and retries

Bulk submissions count as a single request toward your rate limit. On `429 too_many_requests`, use exponential back-off. Do not resubmit partial line sets — the API does not support partial order continuation. Resubmit the full payload.

## Webhooks

Webhook payloads for bulk orders include `resolved_size` on each line, identical in shape to the synchronous response. Pipe these events to your warehouse system to confirm garment dimensions match pick-and-pack expectations.

