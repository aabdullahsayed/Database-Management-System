# Parsing

## Scenario
You accidentally write `SELECT * FROM users WHERE WHERE id = 1;` (a doubled `WHERE` from a bad find-replace) and the database rejects it instantly, before even looking at any data.

## Diagram
```
SQL text -> [Lexer: tokens] -> [Parser: syntax tree] -> [Semantic check: do tables/columns exist?]
"SELECT * FROM users WHERE WHERE id=1"
   -> tokens: SELECT, *, FROM, users, WHERE, WHERE, id, =, 1
   -> parser expects a valid expression after WHERE, gets another WHERE -> SYNTAX ERROR
```

## Problem
Why does this fail before touching any data, and what's the difference between this error and, say, querying a table that doesn't exist?

## Solution
Parsing happens before execution and validates two separate things in sequence: **syntax** (does the query match SQL grammar rules - here it fails: `WHERE WHERE` isn't valid grammar) and, if syntax passes, **semantics** (do the referenced tables/columns actually exist, are types compatible). A doubled `WHERE` fails at the syntax stage - the parser can't even build a valid parse tree, so it never gets to checking whether `users` exists. Querying a nonexistent table (`SELECT * FROM ghost_table`) is syntactically valid SQL but fails at the semantic-check stage instead.

## Takeaway
SQL errors happen in stages - syntax errors (malformed grammar) are caught earliest and cheapest, before the database ever consults the schema catalog or touches actual data; semantic and runtime errors happen later.
