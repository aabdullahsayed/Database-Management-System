# Cardinality

## Scenario
You need to decide: should `department_id` live as a foreign key on the `Employee` table, or should there be a separate join table between `Employee` and `Department`?

## Diagram
```
1-to-Many (one department, many employees):
Department (1) -----------------< (N) Employee
   -> FK "department_id" lives on the "many" side (Employee)

Many-to-Many (an employee can work on many projects, a project has many employees):
Employee (M) >------------------< (N) Project
   -> needs a separate junction table EmployeeProject
```

## Problem
Given "one department has many employees, but an employee belongs to exactly one department" -- where does the foreign key go? Contrast with "employees work on multiple projects."

## Solution
For 1-to-many, the foreign key goes on the "many" side: `Employee.department_id REFERENCES Department(department_id)`. No junction table needed.

For many-to-many (`Employee` <-> `Project`), neither table can hold a single foreign key column, since one employee needs to reference multiple projects and vice versa. This requires a junction table: `EmployeeProject(employee_id, project_id)`.

## Takeaway
Cardinality (1:1, 1:N, M:N) directly determines your schema: 1:N puts an FK on the many side; M:N always needs a separate junction table, regardless of whether the relationship has extra attributes.
