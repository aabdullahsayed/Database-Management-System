# 3. One-to-Many (1:N) Relationships

## The analogy: a teacher and their students

One homeroom teacher has **many** students. Each student has **exactly one** homeroom teacher. The relationship is lopsided — "many" on one side, "one" on the other. This is by far the most common relationship type in real applications: **users have many posts, orders have many line items, folders have many files, authors have many books.**

## Table design

The rule is simple and never changes: **the foreign key always lives on the "many" side.**

```sql
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    name  TEXT
);

CREATE TABLE posts (
    id        SERIAL PRIMARY KEY,
    title     TEXT,
    author_id INTEGER REFERENCES users(id)   -- no UNIQUE here — many posts can share an author_id
);
```

Compare this to the 1:1 table in file 2: the only structural difference is the missing `UNIQUE` constraint. That single constraint is the entire difference between "one passport per person" and "one teacher, many students."

## Analogy for the direction of the arrow

Think of it like a classroom seating chart pinned to the wall: **every desk has exactly one name tag pointing to a teacher** ("this student belongs to Ms. Alice's class"). The teacher's desk doesn't hold a list of 30 student IDs — that would be unwieldy and would need to change every time a student joins or leaves. Instead, each *student* just carries a small tag pointing back to their teacher. That's why the FK sits on the "many" side (`posts.author_id`), not the "one" side.

## Querying it

**Get all posts by a user (the common direction):**
```sql
SELECT * FROM posts WHERE author_id = 4821;
```

**Get a post's author (the reverse direction):**
```sql
SELECT users.name FROM posts
JOIN users ON users.id = posts.author_id
WHERE posts.id = 99;
```

## Cascading — what happens when the "one" side is deleted?

If Alice (a user) is deleted, what happens to her 40 posts? You choose this explicitly:

```sql
-- Option A: delete her posts too
author_id INTEGER REFERENCES users(id) ON DELETE CASCADE

-- Option B: keep the posts, just null out the author
author_id INTEGER REFERENCES users(id) ON DELETE SET NULL

-- Option C: refuse the delete if posts still reference her (the default)
author_id INTEGER REFERENCES users(id) ON DELETE RESTRICT
```

Analogy: if the teacher leaves the school, do her students get expelled too (`CASCADE`), get reassigned to "no teacher" (`SET NULL`), or does the school refuse to let her leave until every student is reassigned (`RESTRICT`)? There's no universally correct choice — it depends on what "deleting a user" should mean in your product.

## The most common real-world 1:N examples

| One side | Many side |
|---|---|
| A customer | their orders |
| An order | its line items |
| A folder | files inside it |
| A blog author | their posts |
| A category | products in that category |
| A YouTube channel | its videos |

Next: [`04-many-to-many.md`](04-many-to-many.md)
