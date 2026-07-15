# Having

## Scenario
You want categories where total revenue exceeds $10,000 this month - filtering on an *aggregated* value, which `WHERE` can't do.

## Problem
Explain why `WHERE SUM(total) > 10000` fails, and write the correct query.

## Solution
```sql
SELECT category, SUM(total) AS category_revenue
FROM orders
JOIN products ON orders.product_id = products.id
GROUP BY category
HAVING SUM(total) > 10000
ORDER BY category_revenue DESC;
```

`WHERE` filters individual rows **before** grouping/aggregation happens; at that point `SUM(total)` doesn't exist yet (it's computed per group). `HAVING` filters **after** grouping, once aggregates are computed - so any condition on an aggregate value must go in `HAVING`, not `WHERE`.

## Takeaway
Filter order in a `GROUP BY` query: `WHERE` (row-level, pre-aggregation) -> `GROUP BY` -> `HAVING` (group-level, post-aggregation). Use `WHERE` to cut rows early for performance, `HAVING` only for conditions on the aggregate itself.
