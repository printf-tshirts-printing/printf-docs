---
title: Sizing and fit
section: guides
last_reviewed: 2026-05-14
owner: devex
covers_endpoints: [POST /v2/orders]
covers_sdks: [printf-js, printf-py, printf-go, printf-java, printf-rb]
---

# Sizing and fit

Every order line carries a `size`. Sizes use standard letter labels:

| Label | Chest (flat) |
|---|---|
| `XS`  | 86 cm  |
| `S`   | 91 cm  |
| `M`   | 97 cm  |
| `L`   | 102 cm |
| `XL`  | 112 cm |
| `2XL` | 122 cm |
| `3XL` | 132 cm |

```json
{
  "designId": "dsn_7fa91c",
  "size": "L",
  "quantity": 250,
  "garmentSku": "tee-classic-black"
}
```

## Picking sizes for an event

The distribution that works for most developer conferences:

| Size | Share |
|---|---|
| S   | 10% |
| M   | 25% |
| L   | 30% |
| XL  | 20% |
| 2XL | 10% |
| 3XL | 5%  |

Order 10% over your headcount. Attendees take a shirt for a colleague who could
not make it, every single time.

## Fit

All garments are a classic unisex cut. If you need fitted or relaxed cuts, talk
to your account manager — it is a per-order arrangement, not an API field.

## Measuring

Chest measurements are flat, laid out, armpit to armpit, doubled. A garment
measured on a body will read differently and is not what our spec sheets use.
