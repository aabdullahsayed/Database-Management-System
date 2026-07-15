# Distributed Databases

## Scenario
Your single-node Postgres instance is maxed out on write throughput even after vertical scaling (bigger machine). Product wants global expansion to Europe and Asia with low-latency reads for local users.

## Diagram
```
Single node:                     Distributed:
[App] -> [1 DB server]           [App US] -> [DB shard/replica US]
  - one machine's limit           [App EU] -> [DB shard/replica EU]
  - single point of failure       [App ASIA] -> [DB shard/replica ASIA]
                                   - horizontal scaling, geo-local latency
                                   - but now: network partitions, consistency tradeoffs
```

## Problem
What two new categories of problems does going distributed introduce that a single-node database never had to deal with?

## Solution
1. **Network partitions**: nodes can become unable to talk to each other even though each is individually healthy - a single-node database never has to reason about "what if part of myself becomes unreachable from another part of myself."
2. **Consistency vs. availability tradeoffs (CAP theorem)**: when a partition happens, you must choose between serving possibly-stale/inconsistent data (favor availability) or refusing to serve some requests until consistency can be guaranteed (favor consistency) - a single-node database, having no partition possibility, never faces this choice.

## Takeaway
Going distributed solves scale and geo-latency problems but introduces fundamentally new failure modes (partitions, consistency tradeoffs) that don't exist in a single-node world - it's not a strictly-better upgrade, it's a different set of tradeoffs.
