# Tuples and Degree

## Scenario
You're reviewing a teammate's migration PR that adds three new columns to the `users` table: `referral_source`, `signup_campaign`, `utm_medium`. They ask: "does this change the degree of the relation?"

## Problem
1. Define "tuple" and "degree" of a relation.
2. Does adding columns change the degree? Does inserting a new user change the degree?

## Solution
1. A **tuple** is a single row - one ordered set of attribute values (e.g. one user record). The **degree** of a relation is the number of attributes (columns) it has, e.g. `users(id, name, email)` has degree 3.
2. Adding the three columns increases the degree from N to N+3 - this is a schema change (DDL) affecting every row. Inserting a new user is a **cardinality** change (number of tuples increases by 1), not a degree change - the shape of each row stays the same.

## Takeaway
Degree = number of columns (schema-level, changes rarely). Cardinality = number of rows (data-level, changes constantly). Confusing the two leads to miscommunication in migration PRs and capacity planning.
