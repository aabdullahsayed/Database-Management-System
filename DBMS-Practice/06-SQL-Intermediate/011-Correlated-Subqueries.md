# Correlated Subqueries

## Scenario
You need each customer's most recent order date, but only using a subquery approach (not a GROUP BY) because you also need several other non-aggregated columns from that specific most-recent order row.

## Solution
```sql
SELECT o1.*
FROM orders o1
WHERE o1.created_at = (
    SELECT MAX(o2.created_at)
    FROM orders o2
    WHERE o2.customer_id = o1.customer_id   -- correlated: references outer query's o1
);
```

Unlike the uncorrelated subquery in the previous problem, this inner query references `o1.customer_id` from the outer query - it must be **re-evaluated once per outer row**, since the answer depends on which customer the outer row belongs to. This is a **correlated subquery**.

**Performance note:** correlated subqueries can be slow (effectively an N+1-style pattern, one subquery execution per outer row) on large tables without a good index on `(customer_id, created_at)`. A window function (`ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC)`) is often a faster, more idiomatic way to express "most recent per group."

## Takeaway
A subquery is "correlated" if it references a column from the outer query - this forces per-row re-evaluation and can be a performance trap at scale; consider window functions as a faster alternative for per-group "latest/top-N" queries.
