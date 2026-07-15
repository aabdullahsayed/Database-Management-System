# Undo

## Scenario
At crash time, transaction T2 had updated `inventory.quantity` from 10 to 9 but never committed. Recovery must reverse that change specifically, using only the log.

## Problem
Given a log entry `T2: quantity 10 -> 9 (no COMMIT record follows before crash)`, describe the undo process.

## Solution
Each log record for an update typically stores both the **old value (before-image)** and the **new value (after-image)**: `T2: update quantity, old=10, new=9`. During undo recovery, the system scans backward through uncommitted transactions' log records and reapplies the **old value**, restoring `quantity` to 10 - as if T2's change never happened. An undo (compensation) log record is itself written, so if the system crashes again *during* recovery, it knows this undo was already performed.

## Takeaway
Undo recovery relies on before-images stored in the log to reverse uncommitted transactions' effects - this is what guarantees atomicity survives a crash, not just a normal `ROLLBACK`.
