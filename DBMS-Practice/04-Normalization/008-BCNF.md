# BCNF (Boyce-Codd Normal Form)

## Scenario
A university scheduling table: `R(student_id, course_id, instructor)`, where each course is taught by exactly one instructor, but an instructor can teach multiple courses. FDs: `{student_id, course_id} -> instructor` and `instructor -> course_id`.

## Problem
Show that R is in 3NF but not in BCNF, and explain the anomaly this causes.

## Solution
Candidate keys: `{student_id, course_id}` (via the first FD) and also `{student_id, instructor}` (since `instructor -> course_id`, this combo determines everything too).

Check `instructor -> course_id`: the left-hand side `instructor` is **not** a superkey (it doesn't determine `student_id`). BCNF requires every FD's left-hand side to be a superkey - this FD violates BCNF, even though it may pass 3NF's weaker "every non-key attribute depends on the whole key" test in some formulations, because `course_id` is part of a candidate key here (3NF has an exception for prime attributes that BCNF does not).

**Anomaly:** if instructor "Dr. Smith" only teaches "CS101" today, that fact (`Smith -> CS101`) is duplicated across every student enrolled in Smith's course. If Dr. Smith switches to teaching "CS102," you must update every one of those rows, or risk inconsistency.

**Decomposition (BCNF):**
```sql
CREATE TABLE InstructorCourse (
    instructor PRIMARY KEY,
    course_id
);
CREATE TABLE Enrollment (
    student_id,
    instructor,
    PRIMARY KEY (student_id, instructor),
    FOREIGN KEY (instructor) REFERENCES InstructorCourse(instructor)
);
```

## Takeaway
BCNF is stricter than 3NF: every determinant must be a superkey, full stop, no exception for attributes that happen to be part of a candidate key. When FDs overlap between candidate keys like this, 3NF can hide anomalies that BCNF catches.
