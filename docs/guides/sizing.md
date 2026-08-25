---
title: Sizing and fit
section: guides
last_reviewed: 2026-08-25
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Sizing and fit

Starting in API version 2.4.0, every order carries an explicit `size_system` and every line carries a `fit`. The API no longer infers a size system from your facility assignment.

## Size systems

| Value | Region |
|-------|--------|
| `US`  | United States |
| `EU`  | Europe (EU/UK) |
| `JP`  | Japan |

Set `size_system` at the order level to apply it to every line. Override it on an individual line when a single order mixes systems (for example, a US-sized tee alongside an EU-sized hoodie).

Line-level `size_system` takes precedence over order-level `size_system`.

## Fit

`fit` is set per line. Accepted values are `unisex`, `mens`, and `womens`. When omitted, the garment's catalog default applies.

## Request example

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

## Response: `resolved_size`

Every line in the response now includes `resolved_size`:

```json
{
  "label": "XL",
  "system": "US",
  "fit": "unisex",
  "chest_cm": 112
}
```

| Field | Type | Description |
|-------|------|-------------|
| `label` | string | Canonical size label as printed on the garment |
| `system` | string | Size system that resolved the label |
| `fit` | string | Fit used for resolution |
| `chest_cm` | integer | Chest measurement in centimetres for this size/fit combination |

## Error codes

| Code | HTTP status | Meaning |
|------|-------------|--------|
| `size_system_ambiguous` | 400 | `size_system` was omitted and your account routes to more than one facility. Add `size_system` to the request. |
| `fit_not_available` | 400 | The requested `fit` is not available for the specified `garmentSku`. |

## SDK support

All five client libraries expose `size_system` and `fit` in the 2.4.0 release:

| SDK | Version |
|-----|---------|
| printf-js | 2.4.0 |
| printf-py | 2.4.0 |
| printf-java | 2.4.0 |
| printf-go | 2.4.0 |
| printf-rb | 2.4.0 |

---

## Migrating to 2.4.0 {#migrating-to-240}

**`size_system` is now required for accounts routing to more than one facility.** Requests that omit it return `400 size_system_ambiguous`. In previous releases the API silently applied the primary facility's default — that behaviour is gone.

**Who is affected:** Any account whose account configuration covers more than one facility. This includes StackFest, Cloud Native Rodeo, ObservaCon, KubeSummit, and ShipItConf on the current keynote tier, and any other multi-facility account. Single-facility accounts continue to receive a deprecation warning in the `X-Printf-Deprecation` response header until 2.5.0, when `size_system` becomes required for everyone.

**What to change:**

```diff
 POST /v2/orders
 {
   "accountId": "acct_stackfest",
+  "size_system": "US",
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
+      "size_system": "US",
+      "fit": "unisex",
       "quantity": 250,
       "garmentSku": "tee-classic-black"
     }
   ]
 }
```

**What happens if you do nothing:** Multi-facility accounts receive `400 size_system_ambiguous` immediately. Single-facility accounts receive a `X-Printf-Deprecation: size_system_required` warning header now and will receive `400 size_system_ambiguous` starting in 2.5.0.

### Migrating saved templates {#migrating-saved-templates}

:::danger
Saved order templates do not appear in the API response and are not automatically updated. Any template created before 2.4.0 carries no `size_system`. On a multi-facility account, every order placed from an unmigrated template will fail with `size_system_ambiguous` immediately.
:::

To migrate a saved template:

1. Open the template in your dashboard or retrieve it via `GET /v2/templates/{templateId}`.
2. Add `size_system` at the template root and on each line where you want to override it.
3. Save with `PUT /v2/templates/{templateId}`.
4. Place a test order from the updated template and confirm `resolved_size` appears on each line in the response.

Repeat for every template linked to a multi-facility account before deploying code that targets 2.4.0.
