# Window Functions

## Scenario
You need each order's total alongside the running total of that customer's spending so far, WITHOUT collapsing rows the way `GROUP BY` would - the report needs every individual order row.

## Solution
```sql
SELECT
    id,
    customer_id,
    total,
    SUM(total) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
    ) AS running_total
FROM orders;
```

Unlike `GROUP BY`, which collapses rows into one row per group, `OVER (...)` computes the aggregate **per row**, using a "window" of related rows (here, all of that customer's orders up to and including the current one, due to the default frame with `ORDER BY`) - while still returning one output row per input row.

## Takeaway
Reach for a window function whenever you need "an aggregate value alongside detail rows" (running totals, per-group ranks, comparisons to a group average) - `GROUP BY` can't do this without losing row-level detail.
