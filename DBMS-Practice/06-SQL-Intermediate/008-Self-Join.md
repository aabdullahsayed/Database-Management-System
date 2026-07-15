# Self Join

## Scenario
Your `employees` table has a `manager_id` column referencing another row in the same table. You need a report showing each employee next to their manager's name.

## Diagram
```
employees
+----+--------+------------+
| id | name   | manager_id |
+----+--------+------------+
| 1  | Alice  | NULL       |   <- top of org, no manager
| 2  | Bob    | 1          |
| 3  | Carol  | 1          |
+----+--------+------------+
```

## Solution
```sql
SELECT
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

A self-join treats one physical table as two logical tables via aliases (`e` for the employee row, `m` for the manager row of the same underlying table). `LEFT JOIN` (not `INNER JOIN`) ensures Alice, who has no manager, still shows up with `manager_name = NULL` instead of being dropped.

## Takeaway
Self-joins are how you query hierarchical or self-referential relationships (employee/manager, category/parent-category, comment/parent-comment) one level at a time - for arbitrary depth hierarchies, you'd reach for a recursive CTE instead.
