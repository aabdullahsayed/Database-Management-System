# Commit

## Scenario
A long-running batch job processes 100,000 rows inside a single transaction and commits only at the very end. It's holding locks and growing a huge undo log the whole time, and other queries are timing out waiting on it.

## Problem
What's wrong with committing only once at the end of a 100,000-row batch job, and how would you fix it?

## Solution
```sql
-- instead of one giant transaction:
BEGIN;
-- ... process 100,000 rows ...
COMMIT;

-- batch in smaller committed chunks:
DO $$
DECLARE batch_size INT := 1000;
BEGIN
  LOOP
    UPDATE big_table SET processed = TRUE
    WHERE id IN (SELECT id FROM big_table WHERE processed = FALSE LIMIT batch_size);
    EXIT WHEN NOT FOUND;
    COMMIT;   -- release locks and shrink transaction log periodically
  END LOOP;
END $$;
```

`COMMIT` is the point where changes become permanent AND locks held by the transaction are released. A single 100,000-row transaction holds every lock it acquired for the entire duration, blocking other queries, and keeps the transaction log/undo space growing the whole time. Committing in smaller batches releases locks incrementally and bounds resource usage.

## Takeaway
Bigger isn't always better for transactions - batch large operations into smaller committed chunks to avoid long lock contention and unbounded transaction log growth, while still keeping each chunk atomic.
