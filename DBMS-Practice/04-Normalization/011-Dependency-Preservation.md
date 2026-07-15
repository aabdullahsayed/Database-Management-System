# Dependency Preservation

## Scenario
You decomposed a table into BCNF to remove anomalies, but now a business rule ("a student can only be assigned one advisor per department") requires a join across three tables to check, and it's slow and easy to bypass.

## Problem
Given `R(student_id, department_id, advisor_id)` with FDs `{student_id, department_id} -> advisor_id` and `advisor_id -> department_id`, a BCNF decomposition splits this into `R1(advisor_id, department_id)` and `R2(student_id, advisor_id)`. Is the original FD `{student_id, department_id} -> advisor_id` preserved?

## Solution
`R1` can enforce `advisor_id -> department_id` directly (it's local to R1). But `{student_id, department_id} -> advisor_id` spans both tables - `student_id` lives in `R2`, `department_id` lives in `R1`. To check this constraint you'd need to JOIN `R1` and `R2` together, which is expensive and easy to skip in application code (someone inserts directly without running the check).

This decomposition achieves BCNF (removes anomalies) but **does not preserve** the original dependency - a known tradeoff: **it is provably impossible in general to guarantee both BCNF and dependency preservation simultaneously.** 3NF, by contrast, always guarantees dependency preservation (at the cost of possibly retaining a small amount of redundancy).

## Solution (practical fix)
When you hit this tradeoff, either: (a) accept 3NF instead of BCNF for this part of the schema, or (b) enforce the cross-table constraint with an application-level check or a trigger, since the schema alone can't guarantee it.

## Takeaway
BCNF eliminates all anomalies but can sacrifice dependency preservation. 3NF is often the pragmatic choice in real systems because it guarantees you can check every business rule locally, table by table.
