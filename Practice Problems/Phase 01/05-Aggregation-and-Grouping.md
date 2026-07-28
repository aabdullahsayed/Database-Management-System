# 5. Aggregation, GROUP BY, HAVING

## Quick Refresher
- Aggregate functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.
- `GROUP BY` collapses rows into groups, one output row per group.
- `HAVING` filters **groups** (after aggregation); `WHERE` filters **rows** (before aggregation).
- Every non-aggregated column in `SELECT` must appear in `GROUP BY` (in strict SQL).

## Practice Problems

### Q1 (Basic). Count the number of employees in each department.
```sql
SELECT dept_id, COUNT(*) AS num_employees
FROM Employees
GROUP BY dept_id;
```

### Q2 (Basic). Find the average salary per department.
```sql
SELECT dept_id, AVG(salary) AS avg_salary
FROM Employees
GROUP BY dept_id;
```

### Q3 (Intermediate). Find departments with more than 5 employees.
```sql
SELECT dept_id, COUNT(*) AS num_employees
FROM Employees
GROUP BY dept_id
HAVING COUNT(*) > 5;
```

### Q4 (Intermediate). Find the highest-paid employee in each department.
```sql
SELECT dept_id, MAX(salary) AS top_salary
FROM Employees
GROUP BY dept_id;
```
**Interview gotcha:** this only gives the *salary value*, not who earns it. To get the employee's name too, you typically need a join back or a window function (see Q7 and Chapter 11) — `SELECT dept_id, emp_name, MAX(salary)...` is **invalid** in strict SQL because `emp_name` isn't grouped or aggregated.

### Q5 (Intermediate). Find departments where the average salary exceeds the company-wide average salary.
```sql
SELECT dept_id, AVG(salary) AS avg_salary
FROM Employees
GROUP BY dept_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM Employees);
```

### Q6 (Advanced/Interview). Explain exactly why this query fails, and how to fix it:
```sql
SELECT dept_id, emp_name, COUNT(*)
FROM Employees
GROUP BY dept_id;
```
**Answer:** `emp_name` is neither aggregated nor included in `GROUP BY`, so the database doesn't know **which** employee's name to show for a department with multiple rows collapsed into one group — it's ambiguous. Fix: either add `emp_name` to `GROUP BY` (which changes the grouping granularity to per-employee, defeating the purpose), or use an aggregate like `MIN(emp_name)`, or restructure using a window function (`ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY ...)`) to pick one specific row per group without collapsing everything.

### Q7 (Advanced/Interview). Find, for each department, the employee(s) with the highest salary — including their name (not just the salary value).
```sql
SELECT dept_id, emp_name, salary
FROM (
    SELECT dept_id, emp_name, salary,
           RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM Employees
) ranked
WHERE rnk = 1;
```
**Why RANK() and not ROW_NUMBER()?** `RANK()` correctly handles **ties** — if two employees in the same department are tied for the highest salary, both get rank 1 and both show up. `ROW_NUMBER()` would arbitrarily pick just one.

### Q8 (Advanced/Interview). What's the performance difference between filtering with `WHERE` before a `GROUP BY` versus filtering with `HAVING` after it, when both *could* technically express the same condition (e.g., filtering on a non-aggregated column)?
**Answer:** If the condition doesn't depend on the aggregate (e.g., `dept_id = 3`), always put it in `WHERE` — it reduces the number of rows **before** the expensive grouping/aggregation work happens, which is much faster. Putting it in `HAVING` instead means the database aggregates *all* rows first and only discards groups afterward — wasted work. `HAVING` should be reserved for conditions that genuinely depend on an aggregate result (like `COUNT(*) > 5`), which can't be evaluated before grouping.
