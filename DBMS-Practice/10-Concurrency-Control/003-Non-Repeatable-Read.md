# Non-Repeatable Read

## Scenario
A report-generation transaction reads a customer's account balance at the start, does some other processing, then reads the balance again at the end to double-check - but gets a *different* value both times, because another transaction committed a change in between.

## Diagram
```
Txn A (long-running report): READ balance = 100 ------------------- READ balance = 60  <- different!
Txn B:                                          UPDATE balance=60; COMMIT
                                                        ^
                            B committed a real, valid change between A's two reads.
```

## Problem
Which isolation level prevents this, and what does it cost?

## Solution
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
`REPEATABLE READ` guarantees that if a transaction reads the same row twice, it will see the same value both times, by taking a consistent snapshot for the duration of the transaction (in snapshot-isolation-based engines like Postgres) - other transactions' commits during that window simply aren't visible to it.

**Cost:** the transaction may now be working with slightly stale data relative to what's currently true in the database, and in some databases (not Postgres's MVCC implementation), this level is achieved via extra locking, which can reduce concurrency/throughput.

## Takeaway
Non-repeatable reads occur under `READ COMMITTED` because each individual statement sees the latest committed data, which can change between statements in the same transaction - `REPEATABLE READ` fixes this by fixing a snapshot for the whole transaction.
