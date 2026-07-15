# Order By

## Problem
The product listing page needs to show the newest products first, and for products created on the same day, cheaper ones should appear first.

## Solution
```sql
SELECT id, name, price, created_at
FROM products
ORDER BY created_at DESC, price ASC;
```

## Solution note
`ORDER BY` supports multiple sort keys evaluated left to right - here it sorts primarily by `created_at` descending, and only uses `price` ascending to break ties among products created on the exact same date/timestamp.

## Takeaway
Multi-column `ORDER BY` is how you express "sort by X, and for ties, sort by Y" - and remember, without an `ORDER BY`, row order is never guaranteed by the database, even if it "looks" consistent in testing.
