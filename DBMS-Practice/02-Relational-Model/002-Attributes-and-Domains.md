# Attributes and Domains

## Scenario
A `price` column was defined as `VARCHAR(20)` "to keep it flexible." Six months later, a report does `SUM(price)` and throws a type error because someone inserted `"19.99 USD"` into the column.

## Problem
Explain what a "domain" is in the relational model and how enforcing it at the schema level would have prevented this bug.

## Solution
A domain is the set of legal, atomic values an attribute can take (e.g. `price` should only ever be a non-negative decimal number). By declaring `price DECIMAL(10,2) NOT NULL CHECK (price >= 0)` instead of a free-text `VARCHAR`, the database itself rejects `"19.99 USD"` at insert time - the constraint is enforced once, centrally, instead of every application needing to remember to validate it.

## Takeaway
Push domain constraints (type, range, nullability) into the schema. "Flexible" text columns just move validation bugs from compile-time/insert-time to runtime, where they're much more expensive to find.
