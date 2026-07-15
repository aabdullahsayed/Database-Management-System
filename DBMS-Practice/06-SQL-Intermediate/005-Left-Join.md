# Left Join

## Scenario
Marketing wants a list of ALL customers and their order count, including customers who have never placed an order (count = 0) - an `INNER JOIN` would drop those customers entirely.

## Solution
```sql
SELECT
    customers.id,
    customers.name,
    COUNT(orders.id) AS order_count
FROM customers
LEFT JOIN orders ON orders.customer_id = customers.id
GROUP BY customers.id, customers.name;
```

Why `COUNT(orders.id)` and not `COUNT(*)`: for customers with no matching orders, the LEFT JOIN produces one row with all `orders.*` columns as `NULL`. `COUNT(orders.id)` correctly counts 0 (since `id` is NULL), while `COUNT(*)` would incorrectly count 1 for that placeholder row.

## Takeaway
Use `LEFT JOIN` to keep all rows from the "left" table regardless of a match, and always aggregate on a column from the right-hand (possibly-NULL) table, never `COUNT(*)`, or you'll miscount zero-match rows as one.
