# Data map and working model

## Source inventory

| Input | System / owner (as represented in pack) | Grain | Join key | Use and known gap |
|---|---|---|---|---|
| `database/flasheats.db` - `orders` | SQLite / core operations | Source order row; a few repeated `order_id`s | `order_id` | Created, promised, pickup, actual delivery and final status. Duplicate rows require an explicit keep rule. |
| `database/flasheats.db` - customers, drivers, restaurants | SQLite / core operations | One entity per ID | respective ID | Available lookup entities; not required in headline KPI. |
| `data/customer_interactions.csv` | CSV / customer support | One contact event | `order_id` | Contact count; does not identify whether a contact was caused by delay. |
| `data/order_interventions.csv` | CSV / dispatch operations | One intervention event | `order_id` | Intervention count / types; timing and assignment are not proof of impact. |
| `api/dispatch_data.json` served from local mock API | HTTP / dispatch | One dispatch record per order | `order_id` | Current ETA and assignment; paginated, with controlled 500 and 429 responses. Not a history of ETA changes. |

The classroom source inventory also lists a second restaurants CSV and restaurant status CSV. They are not needed for this first diagnostic question and are not silently joined. If restaurant prep becomes the selected intervention area, those fields should be profiled and mapped first.

## Workflow model

```text
order_created -> pickup -> actual_delivery
      |             |             |
      +------ promised_eta -------+   (service commitment)

order_id <- support interaction(s)
order_id <- dispatch intervention(s)
order_id <- dispatch API record
```

`order_journey` is one row per cleaned order. Interactions and interventions are aggregated to order-level counts before joining. Dispatch contributes only selected order fields. This avoids a one-to-many join multiplying the order denominator. Cancelled orders have no delivery timestamps in this pack and are outside the completed-delivery KPI.

## Metric definitions

- **Late delivery rate:** share of delivered orders with both promised ETA and actual delivery time present where actual delivery is later than promised ETA. It is a complete-case rate; missing actual timestamps are shown separately.
- **Median order-to-pickup:** median minutes from `created_at` to `pickup_at` for rows with delivery evidence and usable timestamps.
- **Median transit:** median minutes from `pickup_at` to `actual_delivery_at` on the same cohort.
- **Orders with intervention:** count of cleaned order IDs with one or more recorded interventions.

All differences are calculated from the source timestamps as supplied. The pack does not establish time-zone conventions, operational targets for each stage, or causal effects.
