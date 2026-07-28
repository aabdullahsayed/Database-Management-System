# 4. SQL Joins

*(Uses the sample schema: Departments, Employees, Projects, Assignments)*

## Quick Refresher
- **INNER JOIN**: only rows that match in both tables.
- **LEFT JOIN**: all rows from the left table, matched rows from the right (unmatched → NULLs).
- **RIGHT JOIN**: mirror of LEFT JOIN.
- **FULL OUTER JOIN**: all rows from both, matched where possible.
- **CROSS JOIN**: every combination of rows from both tables (Cartesian product).
- **SELF JOIN**: a table joined with itself (e.g., employee-manager relationships).

## Practice Problems

### Q1 (Basic). List each employee's name along with their department name.
```sql
SELECT e.emp_name, d.dept_name
FROM Employees e
INNER JOIN Departments d ON e.dept_id = d.dept_id;
```

### Q2 (Basic). List ALL employees and their department name, including employees with no department.
```sql
SELECT e.emp_name, d.dept_name
FROM Employees e
LEFT JOIN Departments d ON e.dept_id = d.dept_id;
```

### Q3 (Intermediate). Find all departments that currently have zero employees.
```sql
SELECT d.dept_name
FROM Departments d
LEFT JOIN Employees e ON d.dept_id = e.dept_id
WHERE e.emp_id IS NULL;
```
**Key idea:** LEFT JOIN + `WHERE right_table.key IS NULL` is the classic pattern for "find things in A with no match in B."

### Q4 (Intermediate). List each employee alongside their manager's name (self-join).
```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM Employees e
LEFT JOIN Employees m ON e.manager_id = m.emp_id;
```

### Q5 (Intermediate). List employees who are working on projects, showing employee name, project name, and hours — only for employees who have at least one assignment.
```sql
SELECT e.emp_name, p.project_name, a.hours
FROM Employees e
JOIN Assignments a ON e.emp_id = a.emp_id
JOIN Projects p ON a.project_id = p.project_id;
```

### Q6 (Advanced/Interview). What's the difference between `WHERE` and `ON` when filtering in a LEFT JOIN? Give an example where it matters.
**Answer:** Conditions in `ON` are applied **while matching rows** (before the LEFT JOIN "fills in" unmatched left rows with NULLs). Conditions in `WHERE` are applied **after** the join, which can accidentally turn your LEFT JOIN into an INNER JOIN.

Example — get all employees, with project hours **only counting Project 5** (but still show employees without that project):
```sql
-- CORRECT: filter belongs in ON, preserves all employees
SELECT e.emp_name, a.hours
FROM Employees e
LEFT JOIN Assignments a ON e.emp_id = a.emp_id AND a.project_id = 5;

-- WRONG: filtering in WHERE drops employees with no Project 5 assignment entirely
SELECT e.emp_name, a.hours
FROM Employees e
LEFT JOIN Assignments a ON e.emp_id = a.emp_id
WHERE a.project_id = 5;
```

### Q7 (Advanced/Interview). Find department pairs that share at least one employee's manager (i.e., two different departments where an employee in each reports to managers who are in the same department). This tests multi-way joins — write a query and explain your approach.
**Answer (approach):**
```sql
SELECT DISTINCT e1.dept_id AS dept_a, e2.dept_id AS dept_b
FROM Employees e1
JOIN Employees m1 ON e1.manager_id = m1.emp_id
JOIN Employees e2 ON m1.dept_id = e2.manager_id  -- conceptual chain
JOIN Employees m2 ON e2.manager_id = m2.emp_id
WHERE e1.dept_id <> e2.dept_id
  AND m1.dept_id = m2.dept_id;
```
**Explanation:** Chain the self-join twice — once to find each employee's manager, and compare managers' departments across two different employee-department pairs. This kind of multi-hop join is common in "graph-like" relational questions, and it's a good example of why understanding **join order and aliasing** matters — each join adds a new "copy" of the table into the query.

### Q8 (Advanced/Interview). Why can a JOIN with a non-indexed column cause serious performance issues on large tables, and what's the fix?
**Answer:** Without an index, the database must do a full table scan (or nested loop scan) to find matches for each row — O(n·m) in the worst case. Adding an index (typically a B-Tree) on the join column lets the database do a fast lookup (O(log n)) instead, and the query optimizer can choose a more efficient join algorithm (like a Merge Join or Index Nested Loop Join instead of a Hash Join over unindexed data).
