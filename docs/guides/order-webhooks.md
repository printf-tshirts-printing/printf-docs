---
title: Order webhooks
section: guides
last_reviewed: 2026-08-21
owner: platform
covers_endpoints: []
covers_sdks: []
---

# Order webhooks

Printf sends webhook events to your configured endpoint at key points in an order's lifecycle.

## Payload structure

Each event payload wraps a snapshot of the order at the time the event fired. Line items in the payload mirror the structure of the API response, including all fields added in recent releases.

## Size system fields (added in 2.4.0)

As of 2.4.0, each line item in webhook payloads includes `resolved_size`:

```json
{
  "event": "order.confirmed",
  "order": {
    "accountId": "acct_stackfest",
    "size_system": "US",
    "lines": [
      {
        "designId": "dsn_7fa91c",
        "garmentSku": "tee-classic-black",
        "size": "XL",
        "size_system": "US",
        "fit": "unisex",
        "quantity": 250,
        "resolved_size": {
          "label": "XL",
          "system": "US",
          "fit": "unisex",
          "chest_cm": 112
        }
      }
    ]
  }
}
```

`resolved_size` is always present on lines in webhook payloads for orders created on API version 2.4.0 or later. For older orders replayed through webhooks, the field is absent.

## Using resolved_size in downstream systems

If your fulfilment, WMS, or analytics pipeline consumes webhook payloads, `resolved_size.chest_cm` gives you the physical measurement that was used — no lookup table required. Prefer it over the raw `size` label for any size-sensitive logic.

