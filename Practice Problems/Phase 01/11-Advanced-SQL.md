# 11. Advanced SQL — Window Functions & CTEs

## Quick Refresher
- **Window functions** compute a value across a "window" of related rows **without collapsing them** into one row (unlike GROUP BY).
- Syntax: `FUNCTION() OVER (PARTITION BY col ORDER BY col)`.
- `ROW_NUMBER()`: unique sequential number per row (no ties).
- `RANK()`: same rank for ties, but leaves gaps afterward (1, 2, 2, 4).
- `DENSE_RANK()`: same rank for ties, no gaps (1, 2, 2, 3).
- `LAG()`/`LEAD()`: access a previous/next row's value without a self-join.
- **CTE (Common Table Expression)**: `WITH name AS (...) SELECT ...` — a named, reusable temporary result set for the duration of one query. Can be recursive.

## Practice Problems

### Q1 (Basic). What's the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`? Give an example with tied salaries.
**Answer:** For salaries `[90k, 90k, 80k]` ordered descending:
- `ROW_NUMBER()`: 1, 2, 3 (arbitrary tie-break, always unique)
- `RANK()`: 1, 1, 3 (ties share a rank, next rank skips)
- `DENSE_RANK()`: 1, 1, 2 (ties share a rank, no gap)

### Q2 (Basic). Write a query to number each employee's rank by salary within their department.
```sql
SELECT emp_name, dept_id, salary,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS salary_rank
FROM Employees;
```

### Q3 (Intermediate). Find each employee's salary compared to the previous employee hired (ordered by hire_date), using LAG().
```sql
SELECT emp_name, hire_date, salary,
       salary - LAG(salary) OVER (ORDER BY hire_date) AS salary_diff_vs_prev_hire
FROM Employees;
```

### Q4 (Intermediate). Compute a running total of salaries ordered by hire date.
```sql
SELECT emp_name, hire_date, salary,
       SUM(salary) OVER (ORDER BY hire_date
                          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM Employees;
```

### Q5 (Intermediate). Use a CTE to find departments with above-average headcount, then list their employees.
```sql
WITH DeptCounts AS (
    SELECT dept_id, COUNT(*) AS headcount
    FROM Employees
    GROUP BY dept_id
),
AvgHeadcount AS (
    SELECT AVG(headcount) AS avg_hc FROM DeptCounts
)
SELECT e.emp_name, e.dept_id
FROM Employees e
JOIN DeptCounts dc ON e.dept_id = dc.dept_id
CROSS JOIN AvgHeadcount ah
WHERE dc.headcount > ah.avg_hc;
```
**Why a CTE here?** It breaks a complex query into readable, named steps — much easier to reason about (and debug) than deeply nested subqueries.

### Q6 (Advanced/Interview). Write a recursive CTE to find all employees under a given manager, at any depth in the hierarchy (an "org chart" query).
```sql
WITH RECURSIVE OrgChart AS (
    -- Anchor: the starting manager
    SELECT emp_id, emp_name, manager_id, 1 AS depth
    FROM Employees
    WHERE emp_id = 1  -- starting manager's ID

    UNION ALL

    -- Recursive step: find direct reports of everyone already in OrgChart
    SELECT e.emp_id, e.emp_name, e.manager_id, oc.depth + 1
    FROM Employees e
    JOIN OrgChart oc ON e.manager_id = oc.emp_id
)
SELECT * FROM OrgChart;
```
**Key idea:** the recursive CTE runs the "recursive step" repeatedly, each time joining against the **results added in the previous iteration**, until no more new rows are produced — this is how SQL expresses "traverse a tree/graph of unknown depth."

### Q7 (Advanced/Interview). Find the top 2 highest-paid employees in EACH department (not just company-wide) using a window function.
```sql
SELECT emp_name, dept_id, salary
FROM (
    SELECT emp_name, dept_id, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM Employees
) ranked
WHERE rnk <= 2;
```
**Interview note:** this is a very common "top N per group" interview question — the pattern (window function inside a subquery/CTE, then filter on the rank in the outer query) is worth memorizing, since `WHERE` **cannot** directly reference a window function in the same SELECT level (window functions are logically evaluated after WHERE).

### Q8 (Advanced/Interview). What's the performance concern with using a window function like `SUM() OVER (ORDER BY ...)` on a very large, unindexed table, and how would you investigate it?
**Answer:** Window functions require the data to be **sorted** (and partitioned) according to the `OVER` clause before the calculation can run. If there's no index supporting that sort order, the database must perform an expensive in-memory or on-disk sort of the entire relevant dataset before computing the window function — this can be a major bottleneck on large tables. To investigate, run `EXPLAIN ANALYZE` and look for a "Sort" step feeding into the "WindowAgg" step; if it's expensive, consider adding an index matching the `PARTITION BY`/`ORDER BY` columns so the database can read data pre-sorted instead of sorting it on the fly.
