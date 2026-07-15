# Keys Basic

## Scenario
Your `employees` table has `employee_id`, `ssn`, and `email`, all of which happen to be unique per row today. Your manager asks you to pick *one* primary key and explain the tradeoffs of each candidate.

## Problem
Compare `employee_id` (auto-increment int), `ssn`, and `email` as primary key candidates for a production system.

## Solution
- `employee_id` (surrogate key): stable, never changes, small/fast for indexes and foreign keys, but meaningless outside the system.
- `ssn` (natural key): meaningful, but exposing/storing it as a join key everywhere is a security and privacy liability (PII leaking into logs, URLs, foreign keys across many tables) and some countries restrict its use as an identifier.
- `email` (natural key): unique today, but users *can* change their email - if it's the primary key referenced by foreign keys, changing it means cascading updates everywhere, or you must not allow email changes at all.

**Best choice:** surrogate key `employee_id` as primary key; keep `ssn` and `email` as separate `UNIQUE` constrained columns.

## Takeaway
Prefer stable, meaningless surrogate keys (auto-increment/UUID) as primary keys in production systems; use natural/business keys only as `UNIQUE` constraints, not as the key other tables foreign-key against.
