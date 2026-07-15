# Inner Join

## Scenario
You need a report of orders together with the customer's name - but only for orders that have a valid, existing customer (orphaned orders from a data bug should be excluded).

## Diagram
```
orders                customers
+----+------------+   +----+--------+
| id | customer_id|   | id | name   |
+----+------------+   +----+--------+
| 1  | 5          |   | 5  | Alice  |
| 2  | 99         |   | 7  | Bob    |   <- customer_id 99 doesn't exist
+----+------------+   +----+--------+

INNER JOIN result: only order 1 (matches customer 5). Order 2 is dropped.
```

## Solution
```sql
SELECT orders.id, customers.name, orders.total
FROM orders
INNER JOIN customers ON orders.customer_id = customers.id;
```

## Takeaway
`INNER JOIN` returns only rows that have a match on both sides - rows without a match are silently dropped, which is exactly what you want when orphaned/invalid references should be excluded from a report.
