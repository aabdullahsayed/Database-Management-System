# Execution Plan

## Scenario
A query that should be fast (`WHERE customer_id = 5`, and `customer_id` is indexed) is mysteriously slow. You run `EXPLAIN ANALYZE` to see what the database actually did, rather than guessing.

## Solution
```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 5;
```
Example output revealing the problem:
```
Seq Scan on orders  (cost=0.00..18334.00 rows=1 width=64) (actual time=45.2..312.8 rows=1 loops=1)
  Filter: (customer_id = 5)
```
`Seq Scan` (sequential/full table scan) instead of the expected `Index Scan` reveals the index either doesn't exist, isn't being used (e.g. due to a type mismatch like comparing an `int` column to a string literal), or the optimizer's statistics judged a scan cheaper (possible on very small tables, or with stale statistics after a bulk load).

## Takeaway
Never guess why a query is slow - `EXPLAIN` (plan only) or `EXPLAIN ANALYZE` (plan plus actual runtime numbers) shows exactly what the database did, letting you diagnose missing indexes, bad statistics, or unexpected plan choices directly instead of speculating.
