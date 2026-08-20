---
title: Errors
section: api
last_reviewed: 2026-07-28
owner: devex
covers_endpoints: [POST /v2/orders, GET /v2/orders/{id}]
---

# Errors

Every error response carries a stable machine-readable `code` and a human
`message`. Match on `code`. The `message` is prose and we reserve the right to
improve it.

```json
{ "code": "size_unavailable", "message": "Unknown size: XXL" }
```

| Code | Status |
|---|---|
| `size_unavailable` | 400 |
| `art_too_large` | 400 |
| `order_not_found` | 404 |
| `enqueue_failed` | 502 |
| `routing_unavailable` | 503 |

## Warnings

Deprecations arrive as `warnings[]` on a successful response one minor version
before they become errors.

```json
{
  "warnings": [
    { "code": "field_deprecated", "message": "…", "errorsIn": "2.6" }
  ]
}
```

Log them. They are the only advance notice you get.
