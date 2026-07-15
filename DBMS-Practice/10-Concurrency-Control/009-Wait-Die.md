# Wait-Die

## Scenario
Rather than letting deadlocks happen and cleaning up after, you want a scheme that *prevents* deadlocks from forming in the first place, using transaction timestamps (older transactions have priority).

## Problem
Transaction T1 (started earlier, older, lower timestamp) requests a lock held by T2 (started later, younger, higher timestamp). What does Wait-Die dictate, and what if the situation is reversed (T2 requests a lock held by T1)?

## Solution
Wait-Die rule: when Ti requests a lock held by Tj,
- If Ti is **older** than Tj: Ti **waits** (it has priority, so it's allowed to wait for the younger transaction to finish).
- If Ti is **younger** than Tj: Ti **dies** (aborts and restarts later, since it's less "important"/senior than what it's blocked on).

So: T1 (older) requests a lock held by T2 (younger) -> T1 **waits**.
Reversed: T2 (younger) requests a lock held by T1 (older) -> T2 **dies** (aborts, retries later, typically keeping its original timestamp to eventually gain priority and avoid starvation).

This guarantees no cycles can ever form (younger transactions always yield to older ones), so true deadlock is structurally impossible - you trade "some transactions restart" for "no thread ever hangs forever waiting in a cycle."

## Takeaway
Wait-Die is a non-preemptive, timestamp-based deadlock *prevention* scheme (as opposed to detect-and-recover): it guarantees no deadlocks can form, at the cost of occasionally aborting younger transactions rather than letting them wait.
