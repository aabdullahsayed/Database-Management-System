# Two Phase Locking (2PL)

## Scenario
Your DBA explains that the database's locking behavior isn't "grab a lock right when you need it, release right after" - it follows a strict discipline called Two-Phase Locking, and you want to understand why that specific discipline is needed for correctness.

## Diagram
```
Growing Phase              Shrinking Phase
(acquire locks only)       (release locks only)
   |------------------------------|
   ^ transaction start            ^ transaction end (commit/rollback)

   lock(A) lock(B) lock(C) | unlock(A) unlock(B) unlock(C)
                           ^
              once ANY lock is released, NO new locks may be acquired
```

## Problem
Why can't a transaction release lock A right after it's done with A, then later acquire a new lock D, if that seems more efficient?

## Solution
2PL splits every transaction into two strict phases: a **growing phase** where it can only acquire locks (never release), and a **shrinking phase** where it can only release locks (never acquire). Once a transaction releases its first lock, it is forbidden from acquiring any new ones.

This matters because releasing a lock early and then acquiring a new one later can let another transaction sneak in between (reading/writing data based on a partial, inconsistent view of what the first transaction intended) - breaking serializability. 2PL is provably sufficient to guarantee **conflict-serializable** schedules (the concurrent execution is equivalent to *some* serial execution).

## Takeaway
2PL isn't about individual lock efficiency - it's a structural discipline (all acquisitions before any release) that mathematically guarantees serializability, which is why real database concurrency control is built on it (often as "strict 2PL," holding all locks until commit/rollback).
