# Log-Based Recovery

## Scenario
The database server loses power mid-transaction. On restart, it must figure out exactly what was committed, what was in-progress, and restore a consistent state - without ever having "seen" the crash coming.

## Diagram
```
Write-Ahead Log (WAL), append-only, written BEFORE the actual data page is modified:
[LSN 100: T1 update balance 100->80]
[LSN 101: T1 update balance 500->520]
[LSN 102: T1 COMMIT]
[LSN 103: T2 update qty 10->9]
[LSN 104: -- CRASH, T2 never committed --]
```

## Problem
On restart, how does the database use this log to recover, and what happens differently to T1 versus T2?

## Solution
The core rule (**write-ahead logging**): a log record describing a change must be flushed to durable storage *before* the corresponding data page change is. This guarantees the log always has enough information to redo or undo any in-flight change, even if the actual data pages were only partially flushed at crash time.

On restart: T1's log entries end in `COMMIT` (LSN 102), so its changes are **redone** to guarantee durability. T2's log entries have no `COMMIT` record before the crash, so its changes are **undone** (rolled back) to guarantee atomicity - it's as if T2 never started.

## Takeaway
Write-ahead logging is what makes a crash survivable and predictable: log first, data pages later, and recovery replays the log to redo committed work and undo uncommitted work - this is the mechanism underlying both durability and atomicity guarantees.
