# Composite Index

## Scenario
Your query is `WHERE customer_id = ? AND status = 'pending' ORDER BY created_at`. You have separate single-column indexes on `customer_id`, `status`, and `created_at`, but the query is still slow.

## Diagram
```
Composite index on (customer_id, status, created_at):
sorted first by customer_id, then status, then created_at within each group

customer_id | status  | created_at
     5      | pending | 2026-07-01
     5      | pending | 2026-07-03
     5      | shipped | 2026-06-20
     7      | pending | 2026-07-02
```

## Problem
Why don't the three separate single-column indexes combine efficiently, and how does one composite index fix it?

## Solution
The database can typically use only one index efficiently per table access path in most cases (or must do a costly bitmap merge of several index scans) - three separate single-column indexes each narrow the search on their own column but don't help the database jump directly to "customer 5, status pending, sorted by date" as one operation.

```sql
CREATE INDEX idx_orders_customer_status_date
ON orders(customer_id, status, created_at);
```

This composite index is sorted first by `customer_id`, then `status` within each customer, then `created_at` within each status - exactly matching the query's filter+sort pattern, letting the database do one efficient index range scan instead of combining multiple indexes.

**Column order matters**: this index helps `WHERE customer_id = ?`, and `WHERE customer_id = ? AND status = ?`, but does NOT efficiently help a query that filters on `status` alone (skipping the leading column breaks the sorted-prefix advantage).

## Takeaway
Composite indexes should match your most common filter+sort patterns, with columns ordered from most selective/most-commonly-filtered-first to least; a leading-column mismatch makes an otherwise-relevant composite index unusable for that query.
