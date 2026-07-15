# Cross Join

## Scenario
You're generating a report template that needs every combination of `store` x `product` (even for products a store has never sold, to show $0 explicitly), before left-joining in actual sales data.

## Solution
```sql
SELECT stores.name AS store_name, products.name AS product_name
FROM stores
CROSS JOIN products;
```

`CROSS JOIN` produces the Cartesian product: every row of `stores` paired with every row of `products` - if there are 10 stores and 50 products, you get 500 rows, with no `ON` condition. Combine it with a `LEFT JOIN` to actual sales data to build a "matrix" report that explicitly shows zeros for missing combinations, rather than omitting them.

## Solution (danger note)
Accidentally writing `SELECT * FROM orders, order_items` (comma-join with no `WHERE` linking them) is a classic bug that silently produces an unintended cross join, multiplying row counts explosively and corrupting aggregate reports.

## Takeaway
`CROSS JOIN` is occasionally useful intentionally (generating combinations/calendars), but an *accidental* cross join - usually from a forgotten `ON`/`WHERE` clause - is one of the most common and dangerous SQL bugs, since it silently multiplies your row count instead of erroring.
