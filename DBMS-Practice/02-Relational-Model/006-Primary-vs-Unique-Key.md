# Primary vs Unique Key

## Scenario
A teammate asks: "We already have a UNIQUE constraint on `email`. Why do we also need a separate PRIMARY KEY on `id`? Isn't that redundant?"

## Problem
Explain the practical differences between a `PRIMARY KEY` and a `UNIQUE` constraint, and why you'd want both in this scenario.

## Solution
- A table can have only **one** primary key, but **multiple** unique constraints.
- `PRIMARY KEY` implies `NOT NULL` automatically; `UNIQUE` columns can allow one `NULL` (in most databases) unless explicitly marked `NOT NULL`.
- The primary key is typically what other tables' foreign keys reference, and is usually the clustering key that determines physical row storage order (in engines like InnoDB).
- Using `email` as the primary key means every foreign key referencing users stores the (larger, mutable) email string; using `id` as PK + `email UNIQUE` gives you both integrity (no duplicate emails) and a small, stable, immutable join key.

## Takeaway
`PRIMARY KEY` = the one clustering/identity key other tables reference. `UNIQUE` = an additional integrity guarantee on a column that isn't necessarily the row's identity. They serve different jobs even when overlapping in a single-column case.
