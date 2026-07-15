# Non-Clustered Index

## Scenario
Your `orders` table is clustered (physically stored) by `id`, but you frequently query `WHERE customer_id = ?`. You add a secondary index on `customer_id`.

## Diagram
```
Non-clustered index on customer_id:
+-------------+------------+
| customer_id | row pointer|  -> points to the actual row's location
+-------------+------------+   in the clustered structure (keyed by id)
| 5           | -> id=1042 |
| 5           | -> id=1980 |
| 7           | -> id=1101 |
+-------------+------------+
   (a separate structure, doesn't reorder the table itself)
```

## Problem
Explain why a query using this non-clustered index still needs an extra step ("bookmark lookup") compared to querying the clustered key directly.

## Solution
A non-clustered index stores the indexed column's values plus a pointer (or the clustering key) back to the actual row - it does NOT reorder the table itself. So `WHERE customer_id = 5` first finds matching entries in the `customer_id` index (fast, sorted), then must follow each pointer back to fetch the full row from the actual table storage - an extra "bookmark lookup" step per matching row. This is why non-clustered index scans can still be noticeably slower than a clustered index scan for the same number of matching rows, especially if you `SELECT *` instead of only indexed columns.

## Takeaway
A table can have exactly one clustered index (it defines physical storage order) but many non-clustered indexes (separate lookup structures pointing back to the real rows) - non-clustered lookups cost an extra pointer-chase compared to querying the clustered key.
