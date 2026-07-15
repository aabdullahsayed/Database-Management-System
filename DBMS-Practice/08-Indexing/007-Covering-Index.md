# Covering Index

## Scenario
A hot-path query `SELECT customer_id, status FROM orders WHERE customer_id = 5` is still doing extra disk I/O even though `customer_id` is indexed, because it has to fetch `status` from the base table (a bookmark lookup, per the earlier lesson).

## Solution
```sql
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id) INCLUDE (status);
-- (Postgres syntax; some engines just add status to the index key itself)
```

A **covering index** includes every column the query needs (both in `WHERE` and `SELECT`) directly within the index structure itself. Now the database can answer the entire query using only the index - no need to jump back to the base table row at all (an "index-only scan"), eliminating the bookmark lookup entirely.

## Takeaway
When a hot query only touches a small, fixed set of columns, a covering index (or `INCLUDE` clause) lets the database skip the base table entirely and serve the whole query from the index alone - a significant speedup for read-heavy, narrow queries.
