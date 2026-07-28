# 3. SQL Basics — SELECT, WHERE, ORDER BY

*(Uses the sample schema from the README: Departments, Employees, Projects, Assignments)*

## Quick Refresher
- `SELECT columns FROM table WHERE condition ORDER BY column;`
- Logical order of execution: `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`
- `WHERE` filters rows **before** grouping; `HAVING` filters **after** grouping (see Chapter 5).

## Practice Problems

### Q1 (Basic). Get the names and salaries of all employees earning more than 50000.
```sql
SELECT emp_name, salary
FROM Employees
WHERE salary > 50000;
```

### Q2 (Basic). Get all employees hired in the year 2023, ordered by hire date (most recent first).
```sql
SELECT emp_name, hire_date
FROM Employees
WHERE hire_date BETWEEN '2023-01-01' AND '2023-12-31'
ORDER BY hire_date DESC;
```

### Q3 (Basic). Find all employees whose name starts with 'A'.
```sql
SELECT emp_name
FROM Employees
WHERE emp_name LIKE 'A%';
```

### Q4 (Intermediate). Find employees with no manager assigned.
```sql
SELECT emp_name
FROM Employees
WHERE manager_id IS NULL;
```
**Interview note:** always use `IS NULL` / `IS NOT NULL` — `= NULL` never matches anything, because NULL means "unknown," and comparing "unknown = unknown" is itself unknown, not true.

### Q5 (Intermediate). Get the top 3 highest-paid employees.
```sql
SELECT emp_name, salary
FROM Employees
ORDER BY salary DESC
LIMIT 3;
```
**Interview note:** watch for **ties** — `LIMIT 3` might arbitrarily cut off employees tied at 3rd place. If ties should all be included, use `RANK()` or `DENSE_RANK()` instead (see Chapter 11).

### Q6 (Intermediate). Select distinct department IDs that have at least one employee.
```sql
SELECT DISTINCT dept_id
FROM Employees
WHERE dept_id IS NOT NULL;
```

### Q7 (Advanced/Interview). Explain why `SELECT * FROM Employees WHERE salary BETWEEN 40000 AND 60000` might behave unexpectedly with NULLs, and what `BETWEEN` actually expands to.
**Answer:** `BETWEEN a AND b` is shorthand for `salary >= a AND salary <= b`. If `salary` is `NULL`, both comparisons evaluate to `UNKNOWN`, so the row is excluded — NULLs are never "between" anything, which surprises people who expect `BETWEEN` to have special NULL handling.

### Q8 (Advanced/Interview). Write a query to find employees whose salary is an "outlier" — more than 2x the average salary company-wide — without using a subquery in the WHERE clause twice.
```sql
SELECT emp_name, salary
FROM Employees
WHERE salary > 2 * (SELECT AVG(salary) FROM Employees);
```
**Interview note:** the subquery here runs **once** and is treated as a constant scalar during the outer query's execution — it's not recalculated per row, so this is efficient.
