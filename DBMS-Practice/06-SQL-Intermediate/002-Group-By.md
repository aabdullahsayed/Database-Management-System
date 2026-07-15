# Group By

## Scenario
You need revenue broken down per product category, not just a single grand total.

## Problem
Write a query returning total revenue per category.

## Solution
```sql
SELECT category, SUM(total) AS category_revenue
FROM orders
JOIN products ON orders.product_id = products.id
GROUP BY category
ORDER BY category_revenue DESC;
```

Rule: every column in `SELECT` that isn't wrapped in an aggregate function must appear in `GROUP BY` - `category` appears in both, `SUM(total)` is the aggregate. Selecting a non-aggregated, non-grouped column (like `product_name`) here would be an error in strict SQL engines (Postgres) or silently pick an arbitrary value in lenient ones (older MySQL modes) - never rely on the latter.

## Takeaway
`GROUP BY` collapses rows sharing the same value(s) into one row per group; every selected column must either be in the `GROUP BY` list or wrapped in an aggregate function.
