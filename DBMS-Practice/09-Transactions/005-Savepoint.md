# Savepoint

## Scenario
A transaction processes a batch of 5 independent operations (e.g. sending 5 different notifications, each with a DB write for delivery status). If operation #3 fails, you want to undo just #3, not the successful #1 and #2 as well.

## Solution
```sql
BEGIN;

INSERT INTO notifications (user_id, status) VALUES (1, 'sent');
SAVEPOINT after_first;

INSERT INTO notifications (user_id, status) VALUES (2, 'sent');
SAVEPOINT after_second;

-- operation #3 fails (e.g. constraint violation or app-detected error)
ROLLBACK TO SAVEPOINT after_second;   -- undo only #3's partial effects, keep #1 and #2

INSERT INTO notifications (user_id, status) VALUES (4, 'sent');
INSERT INTO notifications (user_id, status) VALUES (5, 'sent');

COMMIT;   -- #1, #2, #4, #5 are all committed; #3 never happened
```

A savepoint is a named marker within a transaction that you can roll back to without discarding the entire transaction - useful for "best effort, skip failures" batch processing while still keeping everything inside one atomic unit if you choose to commit at the end.

## Takeaway
`SAVEPOINT` gives you partial rollback within a single transaction - use it when a multi-step transaction should tolerate individual step failures without throwing away all the prior successful work in that same transaction.
