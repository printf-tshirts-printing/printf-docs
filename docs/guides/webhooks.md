---
title: Webhooks
section: guides
last_reviewed: 2026-06-19
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go]
---

# Webhooks

Subscribe to `order.accepted`, `order.in_production`, `order.shipped` and
`order.cancelled`.

## Verifying the signature

```
X-Printf-Signature: t=1754212364,v1=8d1c…
```

Compute `HMAC-SHA256(secret, "{t}.{rawBody}")` and compare in constant time.
Reject anything where `t` is more than five minutes old, or you have built
yourself a replay endpoint.

## Payload

The payload embeds a snapshot of the order at event time. Replaying an old
webhook shows you what was true then, not now.

:::warning
Payload shape is generated from the orders API spec. New fields can appear
without a major version bump. Parse permissively — ignore fields you do not
recognise rather than rejecting the payload.
:::

## Retries

10s, 1m, 5m, 30m, 2h, 6h, 18h. After the last attempt your endpoint is marked
unhealthy and your CSM is notified.
