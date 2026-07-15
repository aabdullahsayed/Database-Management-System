# Identify Candidate Keys

## Scenario
You inherit this table with no documentation and need to figure out what could safely serve as a primary key before writing a migration.

## Diagram
```
enrollments
+------------+------------+--------------+------------+
| student_id | course_id  | semester     | grade      |
+------------+------------+--------------+------------+
| 1          | CS101      | Fall2025     | A          |
| 1          | CS102      | Fall2025     | B          |
| 2          | CS101      | Fall2025     | A          |
| 1          | CS101      | Spring2026   | B+         |
+------------+------------+--------------+------------+
```

## Problem
A student can retake a course in a different semester, and takes multiple courses per semester. Identify all candidate keys for this table.

## Solution
No single column is unique (student_id repeats, course_id repeats, semester repeats). We need the minimal combination of columns that uniquely identifies a row:
- `(student_id, course_id, semester)` uniquely identifies a row - a student can only have one grade per course per semester. This is a candidate key.
- No subset of these three works alone (e.g. `(student_id, course_id)` fails because of the Fall2025/Spring2026 retake).

**Candidate key:** `{student_id, course_id, semester}` (composite, minimal).

## Takeaway
A candidate key must be both unique (no two rows share the same combination) and minimal (removing any attribute breaks uniqueness). Composite keys are common in junction/enrollment-style tables.
