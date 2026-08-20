---
title: Python
section: sdks
last_reviewed: 2026-05-22
owner: devex
covers_sdks: [printf-py]
covers_endpoints: [POST /v2/orders, GET /v2/orders/{id}]
---

# Python

```bash
pip install printf-sdk
```

```python
from printf import PrintfClient

printf = PrintfClient(api_key=os.environ["PRINTF_API_KEY"])

order = printf.create_order(
    account_id="acct_gowest",
    destination={
        "name": "GoWest Ops",
        "line1": "1 Market St",
        "city": "San Francisco",
        "region": "CA",
        "postal_code": "94105",
        "country_code": "US",
    },
    lines=[
        {"design_id": "dsn_44b201", "size": "M", "quantity": 300, "garment_sku": "tee-classic-white"},
    ],
)

print(order.id, order.facility_id)
```

## Errors

```python
from printf import PrintfApiError

try:
    printf.create_order(...)
except PrintfApiError as exc:
    if exc.code == "size_unavailable":
        ...
    raise
```
