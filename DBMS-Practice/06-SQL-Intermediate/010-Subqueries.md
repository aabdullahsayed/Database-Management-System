# Subqueries

## Scenario
You need to find all customers whose total spending is above the company-wide average order value - a comparison that requires computing the average first.

## Solution
```sql
SELECT id, name
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
    GROUP BY customer_id
    HAVING SUM(total) > (SELECT AVG(total) FROM orders)
);
```

This uses two subqueries: an uncorrelated scalar subquery `(SELECT AVG(total) FROM orders)` computed once, and a subquery in the `IN` clause that returns a set of qualifying customer IDs. The outer query is evaluated against that precomputed set.

## Takeaway
Subqueries let you compose queries in layers - compute an intermediate result first (a scalar, a list, or a derived table), then filter/join against it. They're often easier to read than a single deeply nested query, though sometimes a JOIN or CTE achieves the same thing more efficiently.
