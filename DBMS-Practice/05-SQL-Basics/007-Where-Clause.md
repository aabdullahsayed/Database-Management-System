# Where Clause

## Scenario
You need to find all orders placed in the last 7 days that are either "pending" or "processing", excluding any that were flagged as fraudulent.

## Problem
Write a `WHERE` clause combining a date range, an `IN` list, and a `NOT` exclusion.

## Solution
```sql
SELECT *
FROM orders
WHERE created_at >= now() - INTERVAL '7 days'
  AND status IN ('pending', 'processing')
  AND is_fraudulent = FALSE;
```

Note on precedence: `AND` binds tighter than `OR`, so if you mix them, always use parentheses to make the intended grouping explicit - relying on default operator precedence in a business-critical filter is a common source of subtle bugs (e.g. accidentally returning fraudulent orders because an `OR` grouped wrong).

## Takeaway
`WHERE` clauses with multiple conditions should have explicit parentheses whenever `AND`/`OR` are mixed - never rely on implicit precedence for anything that touches money, fraud, or security filters.
