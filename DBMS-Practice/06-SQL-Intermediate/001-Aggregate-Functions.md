# Aggregate Functions

## Scenario
Finance wants total revenue, average order value, and the highest single order this month.

## Problem
Write one query producing sum, average, count, and max for this month's orders.

## Solution
```sql
SELECT
    SUM(total)   AS total_revenue,
    AVG(total)   AS avg_order_value,
    COUNT(*)     AS order_count,
    MAX(total)   AS largest_order
FROM orders
WHERE created_at >= date_trunc('month', now());
```

Note: `COUNT(*)` counts all rows including NULLs in any column; `COUNT(some_column)` counts only non-NULL values of that column - these can give different answers if `total` is nullable.

## Takeaway
Aggregate functions collapse many rows into one summary row - watch out for `NULL` handling differences between `COUNT(*)` and `COUNT(column)`, and remember `AVG`/`SUM` silently ignore `NULL` values rather than erroring.
