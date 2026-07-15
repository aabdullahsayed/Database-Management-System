# Right Join

## Scenario
A teammate writes `orders RIGHT JOIN customers` and you want to know if this differs from `customers LEFT JOIN orders`, since both seem to "keep all customers."

## Solution
```sql
-- These two are logically equivalent:
SELECT * FROM orders RIGHT JOIN customers ON orders.customer_id = customers.id;
SELECT * FROM customers LEFT JOIN orders ON orders.customer_id = customers.id;
```

`RIGHT JOIN` keeps all rows from the right-hand table (here, `customers`) and NULL-fills unmatched columns from the left table - it's the mirror image of `LEFT JOIN`. `A RIGHT JOIN B` = `B LEFT JOIN A` (with the ON condition unchanged).

## Takeaway
`RIGHT JOIN` is rarely used in practice because it's always rewritable as a `LEFT JOIN` by swapping table order - most style guides prefer `LEFT JOIN` everywhere for consistency and readability, reserving `RIGHT JOIN` for cases where flipping table order would be awkward.
