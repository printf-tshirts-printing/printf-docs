---
title: Sizing and fit
section: guides
last_reviewed: 2026-09-02
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-java, printf-go, printf-rb]
---

# Sizing and fit

Orders API 2.4.0 introduces explicit size systems and fit, and **breaks** any integration that omits `size_system` on an account routed to more than one facility. Read this page before upgrading.

## Size systems

| `size_system` | Chest at XL | Notes |
|---|---|---|
| `US` | 112 cm | Default for most North American facilities |
| `EU` | 107 cm | |
| `JP` | 97 cm | |

The systems are not interchangeable. Sending a bare `XL` without a system and letting the facility default resolve it was always ambiguous; as of 2.4.0 it is an error for any account that can route to more than one facility.

## Fields added in 2.4.0

| Field | Location | Type | Description |
|---|---|---|---|
| `size_system` | order root | `US` \| `EU` \| `JP` | Applies to every line that omits its own `size_system` |
| `size_system` | line | `US` \| `EU` \| `JP` | Overrides the order-level value for this line |
| `fit` | line | `unisex` \| `mens` \| `womens` | Optional. Defaults to `unisex` |
| `resolved_size` | response line | object | See below |
| `X-Printf-Size-System` | response header | string | The system that was applied |

### `resolved_size` object

```json
{ "label": "XL", "system": "US", "fit": "unisex", "chest_cm": 112 }
```

`resolved_size` also appears in webhook payloads for `order.confirmed` and `order.shipped` events.

## Resolution order

When `size_system` is present on a line, that value is used. Otherwise the API walks this chain:

1. `size_system` on the order root
2. `size_system` on the account record
3. The fulfilling facility's default

Step 3 depends on routing, not on the payload. If the account can route to more than one facility, the facility is not known at validation time and the request is rejected with `400 size_system_ambiguous`.

## Error and warning codes

| Code | Status | Meaning | Introduced |
|---|---|---|---|
| `size_system_ambiguous` | `400` | Order omits `size_system` and the account routes to more than one facility | 2.4.0 |
| `size_system_implicit` | Warning header | Order resolved `size_system` from a facility default; single-facility accounts only | 2.4.0 |

`size_system_implicit` becomes a `400` error in **2.6**.

---

## Migrating saved templates {#migrating-saved-templates}

Saved order templates created before 2.4.0 carry no `size_system`. Submitting one unmodified against an account that routes to more than one facility will return `400 size_system_ambiguous`.

**Who is affected:** Any account that (a) uses saved order templates and (b) is configured to route to more than one fulfilment facility. Five Keynote accounts fall into this category: StackFest, Cloud Native Rodeo, ObservaCon, KubeSummit, and ShipItConf.

**What to do:** Add `size_system` at the order level of every saved template. A per-line override is only needed where a single order mixes systems.

```diff
  {
    "accountId": "acct_stackfest",
+   "size_system": "US",
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
        "fit": "unisex",
        "quantity": 250,
        "garmentSku": "tee-classic-black"
      }
    ]
  }
```

**What happens if you do nothing:** The first order submission against a multi-facility account returns `400 size_system_ambiguous` and is rejected. Nothing is printed; nothing is charged. The fix is a one-field addition to the template.

:::warning
Even single-facility accounts should add `size_system` now. The `size_system_implicit` warning becomes a `400` error in **2.6**. An account that moves to a second facility between now and 2.6 will start seeing `size_system_ambiguous` immediately.
:::
