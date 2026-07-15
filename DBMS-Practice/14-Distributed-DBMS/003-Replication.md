# Replication

## Scenario
Your single database instance handling both reads and writes is CPU-bound from read traffic (dashboards, reports) competing with critical write traffic (checkout). You want to scale reads without touching your write path.

## Diagram
```
        writes
          |
          v
      [Primary] ------replicates------> [Replica 1] <- reads (dashboards)
          |                              [Replica 2] <- reads (reports)
       (source of truth)                     ^
                                     read traffic offloaded here,
                                     doesn't compete with writes on Primary
```

## Problem
After adding read replicas, a customer support agent updates an order on the primary, then immediately refreshes a dashboard reading from a replica - and doesn't see the change yet. Why, and what's this called?

## Solution
Most replication setups are **asynchronous**: the primary commits the write immediately and confirms success to the client, then streams the change to replicas afterward, with a small (usually milliseconds, but sometimes more under load) delay. During that window, a replica serves **stale** data - this is called **replication lag**, and the overall consistency model is **eventual consistency** between primary and replicas.

Fixes depend on requirements: route "read-your-own-write" critical paths back to the primary, use synchronous replication for the specific replicas that must be current (at a latency cost), or design the UI to tolerate/communicate brief staleness.

## Takeaway
Replication solves read scalability but introduces replication lag - any workflow that writes then immediately reads and expects to see its own write needs to either read from the primary or use synchronous replication for that path.
