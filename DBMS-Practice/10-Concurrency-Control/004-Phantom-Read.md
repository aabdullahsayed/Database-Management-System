# Phantom Read

## Scenario
A transaction counts "all orders over $1000" twice within the same transaction, expecting the same count for a consistency check - but a new qualifying order got inserted by another transaction in between, and the second count includes it, even under `REPEATABLE READ` in some databases.

## Diagram
```
Txn A: SELECT COUNT(*) FROM orders WHERE total > 1000;  -> 5
Txn B: INSERT INTO orders (total) VALUES (1500); COMMIT;
Txn A: SELECT COUNT(*) FROM orders WHERE total > 1000;  -> 6   <- a "phantom" row appeared
```

## Problem
Why is this different from a non-repeatable read, and which isolation level fully prevents it?

## Solution
A non-repeatable read is about an *existing* row's value changing between reads; a phantom read is about the *set of rows matching a condition* changing between reads - a brand new row satisfying the `WHERE` clause appears (or an existing one disappears). `REPEATABLE READ` guarantees existing rows don't change value mid-transaction but, in the SQL standard's strict definition, doesn't always guarantee the row *set* stays fixed.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```
`SERIALIZABLE`, the strictest isolation level, guarantees the transaction behaves as if it ran completely alone, with no other transaction's inserts/updates/deletes visible at all during its lifetime - eliminating phantom reads entirely (Postgres's implementation of REPEATABLE READ, via MVCC snapshots, actually already prevents phantoms too, but this varies by database engine).

## Takeaway
Phantom reads are about the *row set* changing (inserts/deletes matching your WHERE), not existing row values changing - `SERIALIZABLE` is the only isolation level the SQL standard guarantees eliminates them in all engines.
