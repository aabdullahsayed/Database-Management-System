# Dirty Read

## Scenario
Transaction A is in the middle of processing a refund - it has decremented a customer's balance but NOT yet committed (still validating the refund). Transaction B, running under the weak `READ UNCOMMITTED` isolation level, reads that balance and displays it to the customer - then Transaction A rolls back because the refund failed validation.

## Diagram
```
Txn A: UPDATE balance = balance - 50   (uncommitted)
Txn B:                                  READ balance  <- sees the -50, uncommitted!
Txn A: ROLLBACK   (the -50 never actually happened)
Txn B: ...has already shown the customer a WRONG balance based on data that never existed
```

## Problem
Which isolation level allows this, and what's the minimum fix?

## Solution
This is only possible under `READ UNCOMMITTED`, which allows reading another transaction's uncommitted (potentially-to-be-rolled-back) changes.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```
`READ COMMITTED` (the default in most databases, including Postgres) guarantees a transaction only ever sees data that has actually been committed by other transactions - eliminating dirty reads entirely.

## Takeaway
A dirty read is reading data that might not even exist once the writing transaction finishes - `READ UNCOMMITTED` is rarely appropriate for anything user-facing; `READ COMMITTED` or stricter should be your default.
