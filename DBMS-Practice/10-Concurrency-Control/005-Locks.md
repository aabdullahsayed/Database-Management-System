# Locks

## Scenario
Two transactions both try to update the same row of inventory at the same time ("reserve the last item in stock"). You need to make sure only one of them succeeds in reserving it, without a lost update.

## Diagram
```
Txn A: SELECT quantity FROM inventory WHERE id=1 FOR UPDATE;  -- acquires exclusive row lock
Txn B: SELECT quantity FROM inventory WHERE id=1 FOR UPDATE;  -- BLOCKS, waits for A
Txn A: UPDATE inventory SET quantity = quantity - 1 WHERE id=1;
Txn A: COMMIT;                                                 -- lock released
Txn B: -- now proceeds, sees A's committed change, can act correctly
```

## Problem
Write the reservation logic using an explicit row lock, and explain what `FOR UPDATE` guarantees.

## Solution
```sql
BEGIN;
SELECT quantity FROM inventory WHERE id = 1 FOR UPDATE;
-- application checks: if quantity > 0, proceed
UPDATE inventory SET quantity = quantity - 1 WHERE id = 1;
COMMIT;
```
`SELECT ... FOR UPDATE` acquires an exclusive lock on the selected row(s), so any other transaction trying to lock (via `FOR UPDATE`) or update that same row must wait until the first transaction commits or rolls back - preventing two transactions from both reading `quantity = 1` and both decrementing it to a nonsensical `-1`.

## Takeaway
Explicit row locks (`FOR UPDATE`) are the pessimistic-concurrency answer to lost updates - they trade some throughput (transactions wait instead of running fully in parallel) for correctness guarantees on contended rows.
