# Views

## Scenario
Five different reports all repeat the same complex join of `orders`, `customers`, and `products` with the same filtering logic for "valid, non-cancelled orders." Any schema tweak means updating five queries.

## Solution
```sql
CREATE VIEW valid_orders AS
SELECT
    o.id, o.created_at, o.total,
    c.name AS customer_name,
    p.name AS product_name
FROM orders o
JOIN customers c ON c.id = o.customer_id
JOIN products p ON p.id = o.product_id
WHERE o.status <> 'cancelled';

-- now every report just does:
SELECT * FROM valid_orders WHERE created_at >= now() - INTERVAL '30 days';
```

A view is a saved, named query - it doesn't store data itself (unless it's a materialized view); it re-runs the underlying query every time it's referenced, so it always reflects live data, and the join/filter logic lives in exactly one place.

## Takeaway
Use views to centralize repeated, complex query logic (DRY for SQL) - when many consumers need the "same shape" of data, a view means a schema change or bugfix only needs to happen once.
