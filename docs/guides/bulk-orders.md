---
title: Bulk orders and templates
section: guides
last_reviewed: 2026-04-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py]
---

# Bulk orders and templates

Conference orders are large, repetitive, and placed under time pressure. Saved
templates exist so you are not rebuilding a 2,000-unit payload at midnight.

## Saving a template

Templates are configured in the dashboard, not the API. A template stores the
line structure — designs, sizes, quantities, garment SKUs — and leaves the
destination to be filled in per order.

```json
{
  "accountId": "acct_stackfest",
  "templateId": "tpl_booth_standard",
  "destination": { "...": "..." }
}
```

## What a template does not store

A template stores *what* to print, never *where*. Facility selection happens at
order time, based on the destination you supply.

:::tip
Re-run your template against the sandbox before a large event. A template that
worked last year references design IDs and garment SKUs that may since have been
retired.
:::

## Limits

- 5,000 units per line
- 40 lines per order
- Templates do not expire, but the designs they reference can be archived
