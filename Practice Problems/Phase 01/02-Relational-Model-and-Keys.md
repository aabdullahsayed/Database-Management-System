# 2. Relational Model, Keys & Constraints

## Quick Refresher
- **Primary Key (PK)**: uniquely identifies each row; can't be NULL.
- **Candidate Key**: any column (or set) that *could* be a primary key (unique + minimal).
- **Super Key**: any set of columns that uniquely identifies a row (candidate keys are the *minimal* super keys).
- **Foreign Key (FK)**: a column referencing another table's primary key, enforcing referential integrity.
- **Composite Key**: a primary key made of two or more columns together.
- **Surrogate Key**: an artificial key (like an auto-incrementing ID) with no business meaning.

## Practice Problems

### Q1 (Basic). What's the difference between a primary key and a unique key?
**Answer:** Both enforce uniqueness, but a table can have only **one** primary key (and it can't be NULL), while it can have **multiple** unique keys, and unique keys **can** allow a single NULL value (in most databases).

### Q2 (Basic). What is referential integrity?
**Answer:** A rule ensuring that a foreign key value in one table always matches an existing primary key value in the referenced table (or is NULL, if allowed) — you can't reference a row that doesn't exist.

### Q3 (Intermediate). Given `Employees(emp_id, dept_id)` and `Departments(dept_id, dept_name)`, what happens if you try to delete a department that still has employees?
**Answer:** Depends on the foreign key's `ON DELETE` rule:
- `RESTRICT`/`NO ACTION` (default in many DBs): the delete is **blocked** with an error.
- `CASCADE`: all employees in that department are **also deleted**.
- `SET NULL`: employees' `dept_id` is set to NULL (department becomes "unassigned").

### Q4 (Intermediate). What is a composite key, and when would you use one instead of a surrogate key?
**Answer:** A composite key combines multiple columns to form uniqueness — e.g., `(emp_id, project_id)` in an Assignments table, since neither alone is unique but the pair is. Use composite keys for pure junction tables where the combination naturally *is* the identity; use surrogate keys when you need a simple, stable reference (e.g., for foreign keys elsewhere) or when the "natural" key might change over time.

### Q5 (Intermediate). Explain the difference between a candidate key and a super key with an example.
**Answer:** Suppose a `Students` table has `(student_id, email, ssn)`. All three of `{student_id}`, `{email}`, `{ssn}` are candidate keys (each alone is unique and minimal). `{student_id, email}` is a super key (still unique) but **not** a candidate key, because it's not minimal — you could remove `email` and it would still be unique.

### Q6 (Advanced/Interview). Why might a database designer choose a surrogate key (auto-increment ID) over a natural key (like an email address) for a primary key, even though the natural key is already unique?
**Answer:**
- Natural keys can **change** (a user might update their email) — cascading that change through every foreign key referencing it is expensive and risky.
- Natural keys can be **long or composite** (e.g., a full address), making indexes and joins slower.
- Surrogate keys are **stable and simple** (usually a small integer), which keeps foreign key joins fast.
- Downside: you still need a `UNIQUE` constraint on the natural key to preserve data integrity, and surrogate keys carry no business meaning (harder to debug by eye).

### Q7 (Advanced/Interview). A self-referencing foreign key: `Employees(emp_id PK, manager_id FK -> Employees.emp_id)`. Write SQL to find all employees who have no manager (i.e., top of the hierarchy), and explain a pitfall of self-joins on large tables.
**Answer:**
```sql
SELECT emp_id, emp_name
FROM Employees
WHERE manager_id IS NULL;
```
**Pitfall:** For deep hierarchies (e.g., "find all employees under a given manager, at any depth"), a simple self-join only gets you **one level**. You'd need a **recursive CTE** (`WITH RECURSIVE`) to traverse multiple levels, and on very large/deep hierarchies this can be slow without a good index on `manager_id`.
