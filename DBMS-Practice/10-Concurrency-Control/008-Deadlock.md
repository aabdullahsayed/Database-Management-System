# Deadlock

## Scenario
Two transactions each hold a lock the other one needs, and neither will ever release it - both are stuck waiting forever, and your application's connection pool is silently filling up with hung requests.

## Diagram
```
Txn A: locks row 1 -------- waits for row 2 (held by B)
Txn B: locks row 2 -------- waits for row 1 (held by A)
        ^                          ^
        A can never get row 2 because B has it and is waiting on A.
        B can never get row 1 because A has it and is waiting on B.
        Circular wait -> DEADLOCK
```

## Problem
Given this scenario, what does the database do automatically, and what should your application do?

## Solution
Most databases run a **deadlock detector** that periodically checks the "waits-for" graph for cycles; when found, it picks one transaction as the victim, forcibly aborts it (rolling back its changes) and returns a deadlock error to that client, letting the other transaction proceed.

Application-level fix: catch the deadlock error and **retry the transaction** (usually after a short, jittered backoff) - deadlocks are expected, recoverable events under concurrent load, not fatal bugs. You can also reduce deadlock likelihood by always acquiring locks in a **consistent order** across all transactions (e.g. always lock the lower row ID first) - matching the fix in the Wait-Die lesson.

## Takeaway
Deadlocks are automatically detected and resolved by the database (via a victim abort), but your application must be written to catch that specific error and retry - a transaction that doesn't handle deadlock errors will incorrectly surface as a failed user request instead of transparently retrying.
