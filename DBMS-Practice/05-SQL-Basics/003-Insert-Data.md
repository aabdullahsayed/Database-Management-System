# Insert Data

## Scenario
A batch import job needs to insert 3 new products at once, but must not fail the whole batch if one row violates a constraint (e.g. duplicate SKU) - it should skip conflicts instead.

## Problem
Write an insert that adds multiple rows in one statement and gracefully skips duplicate SKUs instead of erroring out.

## Solution
```sql
INSERT INTO products (sku, name, price)
VALUES
    ('SKU-100', 'Wireless Mouse', 19.99),
    ('SKU-101', 'Mechanical Keyboard', 59.99),
    ('SKU-102', 'USB-C Hub', 24.50)
ON CONFLICT (sku) DO NOTHING;
```

`ON CONFLICT (sku) DO NOTHING` (Postgres "upsert" syntax) tells the database: if a row with that `sku` already exists (assuming `sku` has a unique constraint), skip inserting that row instead of raising an error and aborting the whole statement.

## Takeaway
Batch inserts fail as a whole unit unless you explicitly handle conflicts - a single-row multi-VALUES insert is one statement/one transaction by default, so plan for partial-failure semantics with `ON CONFLICT` (or `INSERT IGNORE` in MySQL) when duplicates are expected.
