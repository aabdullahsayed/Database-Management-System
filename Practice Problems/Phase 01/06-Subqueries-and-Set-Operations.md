# 6. Subqueries & Set Operations

## Quick Refresher
- **Scalar subquery**: returns a single value, usable anywhere an expression is expected.
- **Correlated subquery**: references a column from the outer query — runs (conceptually) once per outer row.
- **Set operations**: `UNION` (dedupes), `UNION ALL` (keeps duplicates, faster), `INTERSECT`, `EXCEPT`/`MINUS`.
- `IN`, `EXISTS`, `ANY`, `ALL` are common ways subqueries interact with the outer query.

## Practice Problems

### Q1 (Basic). Find employees who work in the 'Engineering' department (using a subquery, not a join).
```sql
SELECT emp_name
FROM Employees
WHERE dept_id = (SELECT dept_id FROM Departments WHERE dept_name = 'Engineering');
```

### Q2 (Basic). Find employees who are NOT assigned to any project.
```sql
SELECT emp_name
FROM Employees
WHERE emp_id NOT IN (SELECT emp_id FROM Assignments);
```
**Interview gotcha:** if `Assignments.emp_id` can ever contain `NULL`, `NOT IN` silently returns **zero rows** (because comparing to NULL is UNKNOWN, poisoning the whole NOT IN check). Prefer `NOT EXISTS` to be safe:
```sql
SELECT emp_name
FROM Employees e
WHERE NOT EXISTS (SELECT 1 FROM Assignments a WHERE a.emp_id = e.emp_id);
```

### Q3 (Intermediate). Find employees who earn more than the average salary of their own department (correlated subquery).
```sql
SELECT e.emp_name, e.salary, e.dept_id
FROM Employees e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM Employees e2
    WHERE e2.dept_id = e.dept_id
);
```
**Why correlated?** The inner query references `e.dept_id` from the outer query, so it effectively recalculates per department as each outer row is checked.

### Q4 (Intermediate). Find all departments that have at least one project with a budget over 100000.
```sql
SELECT dept_name
FROM Departments d
WHERE EXISTS (
    SELECT 1 FROM Projects p
    WHERE p.dept_id = d.dept_id AND p.budget > 100000
);
```

### Q5 (Intermediate). Get a combined list of all "high earners" (salary > 100000) and all "senior employees" (hired before 2015), without duplicates.
```sql
SELECT emp_name FROM Employees WHERE salary > 100000
UNION
SELECT emp_name FROM Employees WHERE hire_date < '2015-01-01';
```

### Q6 (Advanced/Interview). What's the performance difference between `IN (subquery)` and `EXISTS (subquery)`, and when should you prefer one over the other?
**Answer:**
- `EXISTS` stops as soon as it finds **one** matching row (short-circuits) — great when the subquery could return many rows, since the engine doesn't need to materialize the whole result set.
- `IN` traditionally needed the full subquery result materialized to compare against — though modern optimizers often rewrite `IN` and `EXISTS` into equivalent semi-join plans, so on many production DBs (Postgres, modern MySQL) performance is now similar.
- Rule of thumb: use `EXISTS` when the subquery result could be large or when NULLs might be present in the subquery's column (safer semantics), use `IN` for small, simple, NULL-free lists for readability.

### Q7 (Advanced/Interview). Write a query to find departments where EVERY employee earns more than 50000 (this requires "for all" logic, which SQL doesn't have directly).
**Answer (approach — invert the logic):**
```sql
SELECT dept_id
FROM Departments d
WHERE NOT EXISTS (
    SELECT 1 FROM Employees e
    WHERE e.dept_id = d.dept_id AND e.salary <= 50000
);
```
**Key insight:** SQL has no native "for all" quantifier, but "for all X, condition holds" is logically equivalent to "there does NOT exist an X where the condition FAILS" — this NOT EXISTS pattern is the standard trick for these "universal quantification" interview questions.

### Q8 (Advanced/Interview). Explain the difference between `UNION` and `UNION ALL`, and why you should default to `UNION ALL` unless you specifically need deduplication.
**Answer:** `UNION` removes duplicate rows from the combined result, which requires an internal sort/hash step to detect duplicates — extra computational cost. `UNION ALL` just concatenates both result sets with no dedup check, making it significantly faster on large result sets. Use `UNION` only when duplicates are actually possible **and** undesired; otherwise `UNION ALL` is the more efficient default.
