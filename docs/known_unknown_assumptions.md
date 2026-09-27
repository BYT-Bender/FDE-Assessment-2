# Data readiness notes

## Known from this run

- SQLite read 1,603 source order rows; the cleaning step retained 1,600 unique orders.
- The raw uniqueness check flagged 6 rows participating in duplicate IDs; the cleaning rule removed 3 extra rows.
- The delivered completion check found 37 delivered orders without `actual_delivery_at`.
- The latest `created_at` was 30 days before the run date, inside the configured 60-day freshness window.
- Dispatch returned 1,600 records. Eight pages were fetched and the reported total matched. The controlled API returned one 500 and one 429; both succeeded on retry.
- There are 1,495 delivered orders with both timestamps used for the late-rate denominator. Late rate on this complete subset is 56.39%.

## Assumptions used

- `promised_eta` is the committed delivery time and `actual_delivery_at` is the completion time.
- A positive actual-minus-promised difference is late; equality or earlier is on time.
- `order_id` is stable across source systems. Order is the reporting grain.
- Only delivered orders with complete promised and actual timestamps belong in the late-rate denominator.
- Deduplication keeps the first row encountered for each `order_id`; the source does not provide a canonical-record marker, so this is a practical reporting rule rather than a verified source-of-truth choice.

## Still unknown

- Whether timestamps share a time zone and whether their recorded meaning is consistent across systems.
- Why 37 delivered records have no completion timestamp and whether missingness is concentrated by restaurant, driver, or date.
- Whether reported ETA values are original promises or later updates.
- Which operational team owns each correction, and whether intervention records are complete.
- Stage-level service targets needed to distinguish a long but normal journey from a slow one.

## Decision boundary

The present output is suitable for choosing where to investigate next. It is not enough to claim that an intervention reduces lateness. Validate event semantics and missingness, then run a small operational pilot with a comparison group before estimating impact.
