# 1. Why Relationships Exist

## The problem: data doesn't live in isolation

Imagine you're building a blogging app. You need to store users and posts. You *could* cram everything into one giant table:

| post_id | post_title | author_name | author_email | author_bio |
|---|---|---|---|---|
| 1 | "Hello World" | Alice | alice@x.com | Loves hiking |
| 2 | "My Trip" | Alice | alice@x.com | Loves hiking |
| 3 | "Recipe" | Bob | bob@x.com | Chef |

**Problem:** Alice's email and bio are duplicated on every post. If Alice updates her bio, you have to update it in every single row. If you typo it once, your data is now inconsistent. This is called a **data anomaly**, and it's the core reason relational databases split data into separate tables and *link* them — instead of duplicating.

## Analogy: a filing cabinet vs. a library card catalog

Think of the bad table above as photocopying Alice's entire ID card and stapling it to every single document she's ever written. Wasteful, and if her address changes, you have to find and update every staple.

A **relational** approach is more like a library:
- There's one **card** for Alice in the "People" drawer (the `users` table) — her info lives in exactly one place.
- Each **book** in the "Books" section (the `posts` table) has a little tag that just says "Author: card #4821" — a **reference**, not a copy.

That reference is a **foreign key**.

## Primary keys and foreign keys

- **Primary key (PK)** — a unique ID for a row in its own table. Alice's library card number. No two people share a card number.
- **Foreign key (FK)** — a column in one table that stores another table's primary key, creating a link.

```sql
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,     -- Alice's card number
    name  TEXT,
    email TEXT
);

CREATE TABLE posts (
    id        SERIAL PRIMARY KEY,
    title     TEXT,
    author_id INTEGER REFERENCES users(id)   -- the "tag" pointing back to a user
);
```

Now Alice's info exists once. Every post just holds a small number (`author_id`) pointing back to her.

## The three shapes a relationship can take

Every relationship between two tables boils down to one of three shapes, based on **how many rows on each side can point to how many rows on the other side**:

| Type | Analogy | Example |
|---|---|---|
| **One-to-one (1:1)** | A person and their passport — each person has exactly one, each passport belongs to exactly one person | `users` ↔ `user_profiles` |
| **One-to-many (1:N)** | A teacher and their students — one teacher, many students, but each student has one homeroom teacher | `users` ↔ `posts` |
| **Many-to-many (N:M)** | Students and courses — a student takes many courses, a course has many students | `students` ↔ `courses` |

The next three files walk through each shape in depth, with the analogy carried all the way to the actual table design.

## Why this matters for ORMs

An ORM (Object-Relational Mapper) is a library that lets your code treat rows as objects and relationships as *properties* on those objects — `alice.posts` instead of a manual SQL join. But an ORM can only generate the right SQL if **you first tell it what shape the relationship is**. Every ORM configuration you'll write — `hasOne`, `hasMany`, `ManyToManyField`, `belongsTo` — is just you declaring "this is a 1:1" or "this is a 1:N" or "this is an N:M" in that framework's syntax.

So understanding the three shapes *first*, independent of any ORM, makes every ORM's relationship syntax look obvious instead of magical.

Next: [`02-one-to-one.md`](02-one-to-one.md)
