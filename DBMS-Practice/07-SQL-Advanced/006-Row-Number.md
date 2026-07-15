# Row Number

## Scenario
Classic interview/production problem: "delete duplicate rows, keeping only the earliest one" for a `customer_emails` table that accidentally allowed duplicate emails before a `UNIQUE` constraint was added.

## Solution
```sql
WITH ranked AS (
    SELECT
        id,
        email,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at ASC) AS rn
    FROM customer_emails
)
DELETE FROM customer_emails
WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

`ROW_NUMBER()` assigns a strictly increasing, unique number (1, 2, 3, ...) within each partition, with **no ties possible** - even if two rows have identical `created_at`, they still get distinct row numbers (tie-breaking is arbitrary unless you add more `ORDER BY` columns). Keeping `rn = 1` per email and deleting the rest removes duplicates while keeping the earliest.

## Takeaway
`ROW_NUMBER()` is the go-to tool for deduplication and "top-N per group" queries; unlike `RANK`/`DENSE_RANK`, it never produces ties - always exactly one row per number per partition.
