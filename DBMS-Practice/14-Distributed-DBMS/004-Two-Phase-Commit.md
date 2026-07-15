# Two Phase Commit (2PC)

## Scenario
A single business transaction ("place an order") needs to atomically commit changes across two separate databases: the `orders` database and a separate `inventory` database (different services, different DBs) - both must succeed together, or neither should.

## Diagram
```
Coordinator                     Participant A (orders DB)   Participant B (inventory DB)
    |-- PREPARE ---------------------->|                          |
    |-- PREPARE ------------------------------------------------->|
    |<-- VOTE YES (ready to commit) ---|                          |
    |<-- VOTE YES (ready to commit) --------------------------------|
    |-- COMMIT ------------------------>|                          |
    |-- COMMIT -------------------------------------------------->|
    (only after BOTH vote yes does the coordinator tell BOTH to actually commit)
```

## Problem
What happens if Participant B votes "no" (e.g. insufficient inventory) during the prepare phase?

## Solution
If any participant votes "no" (or fails to respond) during the **prepare phase**, the coordinator sends `ABORT` to every participant instead of `COMMIT`, and each rolls back whatever tentative/prepared state it had - guaranteeing all-or-nothing across both databases, just like a single-database transaction guarantees atomicity across multiple rows.

**Known weakness**: if the coordinator itself crashes after participants voted "yes" but before sending the final `COMMIT`/`ABORT`, participants are left **blocked**, holding locks indefinitely, unsure whether to commit or abort - this "blocking problem" is why 2PC, while correct, is often avoided at large scale in favor of alternatives (sagas, eventual consistency patterns) that trade strict atomicity for availability.

## Takeaway
2PC gives you atomic commits across multiple independent databases via a prepare-then-commit handshake, but its coordinator-crash blocking problem is a real operational risk - many distributed systems intentionally avoid 2PC in favor of compensating-transaction (saga) patterns for this reason.
