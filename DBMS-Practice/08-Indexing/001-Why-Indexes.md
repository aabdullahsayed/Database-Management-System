# Why Indexes

## Scenario
A `SELECT * FROM users WHERE email = 'x@example.com'` query, which used to be instant, now takes 3 seconds after the `users` table grew to 10 million rows.

## Diagram
```
Without index: full table scan
[row1][row2][row3]...[row 9,999,999][row 10,000,000]
  compare each row's email to 'x@example.com' one by one -> O(n)

With index on email: B-Tree lookup
        [ M ]
       /     \
    [D-L]   [N-Z]
    /  \      \
  ...  ...   [x@example.com -> row 4,213,009]  -> O(log n)
```

## Problem
Explain why adding an index fixes this, and what it costs you in return.

## Solution
```sql
CREATE INDEX idx_users_email ON users(email);
```
Without an index, the database has no way to know where a matching row is, so it must scan every row (full table scan, O(n)). An index maintains a sorted structure (typically a B+Tree) mapping `email` values to row locations, letting the database jump almost directly to the match (O(log n)).

**Cost:** every `INSERT`/`UPDATE`/`DELETE` on `users` must now also update the index, making writes slightly slower, and the index consumes additional disk space. Indexes are a read-speed-for-write-speed-and-storage tradeoff.

## Takeaway
Indexes turn O(n) lookups into roughly O(log n), but they aren't free - only index columns that are actually queried frequently (in `WHERE`, `JOIN`, `ORDER BY`), since every unnecessary index slows down writes.
