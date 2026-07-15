# CAP Theorem

## Scenario
Your distributed order database experiences a network partition between the US and EU data centers. Orders are still coming in on both sides. You must decide, right now, what the system does.

## Diagram
```
        Consistency
           /\
          /  \
         /    \
        /      \
Availability --- Partition Tolerance

CAP theorem: during an actual network Partition (which WILL happen eventually
in any distributed system), you must choose between Consistency and Availability.
You cannot have all three simultaneously during a partition.
```

## Problem
Given the partition, what does choosing "CP" (consistency + partition tolerance) mean for the order system's behavior, versus choosing "AP" (availability + partition tolerance)?

## Solution
**CP choice**: the system refuses to accept new orders on the minority/disconnected side of the partition (or on both sides, depending on design) until connectivity is restored and it can guarantee a consistent view - trading availability (some legitimate order requests get rejected/delayed) for guaranteed consistency (no conflicting/duplicate order state once resolved).

**AP choice**: both sides keep accepting orders independently during the partition, staying available - but this risks conflicts (e.g. the same inventory item oversold on both sides) that must be reconciled after the partition heals, trading strict consistency for uptime.

For an order/payment system, most teams lean CP for the payment-critical path (better to briefly reject an order than double-charge or oversell) while choosing AP for less critical paths (e.g. product browsing/search can serve slightly stale data during a partition without real harm).

## Takeaway
CAP isn't "pick one forever" - it's a per-partition-event decision, and mature systems often apply CP to their critical-consistency paths (payments, inventory) and AP to their tolerant-of-staleness paths (browsing, search) within the same overall application.
