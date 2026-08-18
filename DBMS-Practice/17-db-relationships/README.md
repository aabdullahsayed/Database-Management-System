# Database Relationships & ORMs — A Guide With Analogies

Understanding how tables relate to each other, and how ORMs (SQLAlchemy, Prisma, Django ORM, Mongoose) let you work with those relationships as objects instead of raw SQL joins.

## Contents

1. [`01-why-relationships-exist.md`](01-why-relationships-exist.md) — the problem relationships solve, primary/foreign keys explained with an analogy
2. [`02-one-to-one.md`](02-one-to-one.md) — 1:1 relationships (the "marriage certificate" model)
3. [`03-one-to-many.md`](03-one-to-many.md) — 1:N relationships (the "classroom" model)
4. [`04-many-to-many.md`](04-many-to-many.md) — N:M relationships (the "wedding guest list" model)
5. [`05-orm-mapping.md`](05-orm-mapping.md) — how each relationship type looks in SQLAlchemy, Django ORM, Prisma, and Mongoose
6. [`06-full-app-example.md`](06-full-app-example.md) — a real app schema (blog platform) using all three relationship types together, in both raw SQL and ORM code

## The one-sentence version

> A relationship is just "which row points to which other row" — and an ORM is a translator that lets you say `user.posts` in code instead of writing `SELECT * FROM posts WHERE user_id = ?` by hand every time.

## How to read this

If you're brand new to relational databases: read 1 → 4 in order first, then 5.
If you already know SQL joins and just want to see the ORM syntax: skim 1, then jump to 5 and 6.
