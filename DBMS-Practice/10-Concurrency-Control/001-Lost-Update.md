# Lost Update

## Scenario
Two customer support agents both open the same support ticket at the same time to update its status. Agent A sets it to "resolved," Agent B (who loaded the page a second earlier, before A's change) sets it to "escalated" - and A's "resolved" update vanishes without anyone noticing.

## Diagram
```
Time -->
Agent A: READ status='open' ------------------- WRITE status='resolved' (COMMIT)
Agent B: READ status='open' --- WRITE status='escalated' (COMMIT) ------
                                        ^
                          B's write happens using a stale read,
                          then A's write overwrites it silently.
Final state: 'resolved' -- but nobody knows 'escalated' was ever set. LOST.
```

## Problem
Identify why this happens and how to prevent it in SQL.

## Solution
Both transactions read the same initial value, then each writes based on that stale read, and whichever commits last silently overwrites the other with no error - a classic **lost update**.

Fix with optimistic concurrency (version column):
```sql
UPDATE tickets
SET status = 'resolved', version = version + 1
WHERE id = 42 AND version = 7;   -- fails (0 rows updated) if someone else already changed it
```
If `0` rows are updated, the application knows a conflict occurred and can reload/retry instead of silently succeeding on stale data.

## Takeaway
Lost updates happen when two transactions read-then-write the same data without coordination - use optimistic locking (version columns) or explicit row locks (`SELECT ... FOR UPDATE`) to detect or prevent it.
