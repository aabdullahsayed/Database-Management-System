# Attribute Closure

## Scenario
You're deciding whether `{student_id, course_id}` is enough information to derive every other column in the `enrollments` table, before using it as a key in a migration.

## Problem
Given `R(student_id, course_id, student_name, course_name, grade)` and FDs:
```
student_id -> student_name
course_id  -> course_name
{student_id, course_id} -> grade
```
Compute the attribute closure of `{student_id, course_id}`, written `{student_id, course_id}+`.

## Solution
Start: `closure = {student_id, course_id}`

1. `student_id -> student_name` applies (student_id is in closure) -> add `student_name`.
   `closure = {student_id, course_id, student_name}`
2. `course_id -> course_name` applies -> add `course_name`.
   `closure = {student_id, course_id, student_name, course_name}`
3. `{student_id, course_id} -> grade` applies -> add `grade`.
   `closure = {student_id, course_id, student_name, course_name, grade}`

No more FDs apply. Final closure = **all attributes of R**.

Since `{student_id, course_id}+` = all attributes of R, `{student_id, course_id}` is a **superkey** (and, since neither attribute alone determines all others, it's a candidate key).

## Takeaway
Attribute closure is the mechanical algorithm for answering "does this set of columns determine everything else?" - repeatedly apply FDs whose left side is already in your closure set until nothing new is added.
