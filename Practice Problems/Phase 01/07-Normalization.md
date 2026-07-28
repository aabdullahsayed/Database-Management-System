# 7. Normalization (1NF → BCNF → 4NF)

## Quick Refresher (Plain English)
Normalization means organizing tables to **avoid storing the same fact in multiple places**, which prevents weird bugs when you update data.

- **1NF (First Normal Form)**: every column holds a single (atomic) value — no lists or repeating groups crammed into one cell.
- **2NF**: 1NF + every non-key column depends on the **whole** primary key, not just part of it (only relevant for composite keys).
- **3NF**: 2NF + no non-key column depends on **another non-key column** (no "transitive" dependency).
- **BCNF (Boyce-Codd Normal Form)**: a stricter version of 3NF — every determinant (a column that determines another) must be a candidate key.
- **4NF**: BCNF + no multi-valued dependencies (avoid combining two independent multi-valued facts in one table).

## The Three Anomalies Normalization Prevents
- **Update anomaly**: changing one fact requires updating multiple rows (and risking inconsistency if you miss one).
- **Insert anomaly**: you can't add a new fact without also having unrelated data available.
- **Delete anomaly**: deleting a row accidentally destroys unrelated data too.

## Practice Problems

### Q1 (Basic). Is this table in 1NF? Why or why not?
| student_id | name | courses |
|---|---|---|
| 1 | Alice | Math, Physics |

**Answer:** No — the `courses` column holds **multiple values** in one cell, violating atomicity. Fix: split into a separate `Student_Courses` table with one row per (student, course) pair.

### Q2 (Basic). Given `OrderItems(order_id, product_id, product_name, quantity)` where the primary key is `(order_id, product_id)`, is this in 2NF?
**Answer:** No. `product_name` depends only on `product_id` (part of the composite key), not on the whole key `(order_id, product_id)` — this is a **partial dependency**, which violates 2NF. Fix: move `product_name` into a separate `Products(product_id, product_name)` table.

### Q3 (Intermediate). Given `Employees(emp_id, dept_id, dept_name)` where `dept_name` depends on `dept_id` (not directly on `emp_id`), what normal form is violated, and how do you fix it?
**Answer:** This violates **3NF** — `dept_name` transitively depends on `emp_id` through `dept_id` (a non-key column determining another non-key column). Fix: split into `Employees(emp_id, dept_id)` and `Departments(dept_id, dept_name)`.

### Q4 (Intermediate). Why does normalization sometimes hurt read performance, and what's the common real-world compromise?
**Answer:** Normalized data is spread across more tables, so retrieving a "full picture" (e.g., an order with product names) requires more `JOIN`s, which cost more at query time. The common compromise is **denormalization** for read-heavy systems — deliberately duplicating some data (e.g., storing `product_name` directly on `OrderItems` too) to avoid joins, accepting the small risk/cost of keeping duplicates in sync (often mitigated via application logic, triggers, or accepting eventual consistency).

### Q5 (Advanced/Interview). Explain the difference between 3NF and BCNF with an example where a table is in 3NF but NOT in BCNF.
**Answer:** Consider `Bookings(student_id, course_id, instructor)` where:
- Each `(student_id, course_id)` is unique (composite key).
- Each `course_id` has exactly one `instructor` (a course only has one instructor).
- But each `instructor` can teach only one `course_id` (in this specific hypothetical school).

Here `instructor → course_id` (instructor determines course) even though `instructor` isn't a candidate key. This table can still be in 3NF (no transitive dependency on a *non-key* column caused by another *non-key* column relative to the *primary* key), but it violates BCNF because `instructor` is a determinant that isn't a candidate key. Fix: split into `Course_Instructor(instructor, course_id)` and `Bookings(student_id, instructor)`.

### Q6 (Advanced/Interview). What is a multi-valued dependency, and how does 4NF address it? Give an example.
**Answer:** A multi-valued dependency happens when one key is associated with multiple **independent** sets of values. Example: `Employee_Skills_Languages(emp_id, skill, language)` where an employee's skills and spoken languages are totally independent of each other, but stored in the same table — this forces you to store **every combination** of skill × language per employee, creating redundant rows. 4NF fixes this by splitting into two separate tables: `Employee_Skills(emp_id, skill)` and `Employee_Languages(emp_id, language)`, since the two facts don't actually relate to each other.

### Q7 (Advanced/Interview). In a real interview, when would you argue AGAINST fully normalizing a schema?
**Answer:** For **analytics/reporting/data warehouse** systems (OLAP), where read performance and simple queries matter more than avoiding update anomalies (data is often loaded in batches, not updated in place) — a **denormalized star schema** (fact table + dimension tables) is standard and intentional. Also for **high-read, low-write** services where join costs at scale outweigh the storage/consistency benefits of full normalization, especially when caching or read-replicas can absorb some of the consistency risk.
