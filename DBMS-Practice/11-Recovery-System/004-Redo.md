# Redo

## Scenario
Transaction T1 committed successfully right before the crash, but its data page changes hadn't yet been flushed from the buffer pool (in-memory cache) to disk - the OS/hardware failure happened before that flush completed.

## Problem
Given the log shows `T1: balance 100->80, COMMIT`, but the on-disk data still shows `balance=100`, how does redo recovery fix this?

## Solution
Because T1's log record includes `COMMIT`, the database knows this transaction's effects MUST be durable - the client was told "success." Redo recovery scans forward through the log and **reapplies the after-image** (`balance=80`) to the actual data page, regardless of whether that page was flushed before the crash. This is safe and correct precisely because of write-ahead logging: the log entry existed durably before the crash, even though the data page itself didn't get updated in time.

## Takeaway
Redo recovery guarantees durability: "if I told the client it committed, that change WILL exist after I recover," even if the underlying data page write hadn't physically happened yet at crash time - the log is the source of truth, the data pages just eventually catch up.
