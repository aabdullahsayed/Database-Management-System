# 4. Many-to-Many (N:M) Relationships

## The analogy: students and courses

A student takes many courses. A course has many students. Neither side is "the owner" of the other — it's a genuine many-on-both-sides relationship. Other classic examples: **actors and movies** (an actor is in many movies, a movie has many actors), **posts and tags**, **products and orders**.

## Why you can't just add a foreign key this time

With 1:N, you could put a single `author_id` column on `posts` because each post has *only one* author. But a course has *many* students — you can't fit a list of student IDs into one column of the `courses` table (and even if your database technically allows an array column, it breaks the ability to efficiently query, index, and join — more on this below).

## The solution: a junction table (a.k.a. join table)

You introduce a **third table** whose entire job is to record individual pairings.

```sql
CREATE TABLE students (
    id   SERIAL PRIMARY KEY,
    name TEXT
);

CREATE TABLE courses (
    id    SERIAL PRIMARY KEY,
    title TEXT
);

-- The junction table: one row = one (student, course) pairing
CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students(id),
    course_id  INTEGER REFERENCES courses(id),
    enrolled_on DATE DEFAULT CURRENT_DATE,
    PRIMARY KEY (student_id, course_id)   -- prevents enrolling in the same course twice
);
```

## Analogy: the wedding guest list vs. the seating chart

Picture two lists at a wedding: the guest list (`students`) and the table list (`courses`). Now imagine a **third sheet of paper** — the seating chart — where each line just says "Guest #12 sits at Table #4." That seating chart is the junction table: it doesn't describe guests or tables, it only records *pairings* between them. Add a new pairing, and one line is added to the chart — nothing about the guest list or table list needs to change.

Every N:M relationship, no matter how complex, decomposes into exactly this: two 1:N relationships pointing into one junction table.

```
students ──1:N──► enrollments ◄──N:1── courses
```

`enrollments` is "many" relative to both `students` and `courses` — which is exactly how you get a many-to-many out of two simple one-to-many relationships.

## Querying it

**All courses a student is taking:**
```sql
SELECT courses.title
FROM courses
JOIN enrollments ON enrollments.course_id = courses.id
WHERE enrollments.student_id = 42;
```

**All students in a course:**
```sql
SELECT students.name
FROM students
JOIN enrollments ON enrollments.student_id = students.id
WHERE enrollments.course_id = 7;
```

## Junction tables can carry their own data

This is the detail people miss: the junction table isn't just two foreign keys — it's a real table and can hold data that only makes sense *about the pairing itself*, not about either side alone. `enrolled_on` (above) is a good example; a `grade` column would be another. Think of the seating chart again: "Guest #12, Table #4, dietary preference: vegetarian" — the dietary preference for *that seat* isn't a property of the guest in general (they might sit differently at another event) or of the table — it's a property of the pairing.

## Common real-world N:M examples

| Side A | Side B | Junction table |
|---|---|---|
| Students | Courses | `enrollments` |
| Posts | Tags | `post_tags` |
| Actors | Movies | `cast_members` |
| Products | Orders | `order_items` (often carries `quantity`, `price_at_purchase`) |
| Users | Roles | `user_roles` |

Next: [`05-orm-mapping.md`](05-orm-mapping.md) — how all three shapes are expressed inside real ORMs.
