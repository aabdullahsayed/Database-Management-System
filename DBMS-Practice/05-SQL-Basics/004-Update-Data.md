# Update Data

## Scenario
A pricing bug caused every product in the "Electronics" category to be off by a 10% markup that needs correcting - but you must NOT touch products in other categories, and you want to verify the blast radius before running it in production.

## Problem
Write a safe update that only touches "Electronics" products, and explain the safety step you'd take before running it against production.

## Solution
```sql
-- Step 1: verify blast radius first
SELECT COUNT(*) FROM products WHERE category = 'Electronics';

-- Step 2: run inside a transaction so you can roll back if the count looks wrong
BEGIN;

UPDATE products
SET price = ROUND(price / 1.10, 2)
WHERE category = 'Electronics';

-- Step 3: inspect a sample before committing
SELECT id, name, price FROM products WHERE category = 'Electronics' LIMIT 10;

COMMIT;   -- or ROLLBACK; if something looks wrong
```

## Takeaway
Never run a production `UPDATE` without a `WHERE` clause you've verified with a matching `SELECT COUNT(*)` first, and wrap it in a transaction so you can `ROLLBACK` if the affected row count or sampled values look wrong.
