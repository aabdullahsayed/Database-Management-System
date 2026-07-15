# Checkpoint

## Scenario
Your database has been running for 6 months without a restart. The transaction log has grown to 500GB. A crash now would require replaying the *entire* 500GB log from the very beginning to recover - unacceptably slow.

## Diagram
```
Log:  [.......................][CHECKPOINT][...........][CRASH]
                                     ^
                     recovery only needs to start from HERE,
                     not from the very beginning of the log
```

## Problem
What does a checkpoint do, and how does it bound recovery time?

## Solution
A checkpoint periodically flushes all currently-dirty (modified but not-yet-persisted) data pages to disk and records a marker in the log noting "everything before this point is safely on disk." Recovery then only needs to replay the log **from the most recent checkpoint forward**, not from the beginning of time - since everything before the checkpoint is already guaranteed durable.

```sql
CHECKPOINT;   -- e.g. in Postgres, can be issued manually or happens automatically
```

## Takeaway
Checkpoints exist purely to bound recovery time - without them, recovery time grows unboundedly with system uptime; periodic checkpointing keeps "how much log do we have to replay after a crash" roughly constant.
