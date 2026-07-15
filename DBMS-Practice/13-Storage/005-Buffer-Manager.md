# Buffer Manager

## Scenario
Your database server has 16GB of RAM but a 500GB dataset. Every query obviously can't have its data purely in memory - you want to understand how the database decides what stays cached and what gets evicted.

## Diagram
```
Application/Query Engine
        |
        v
+-------------------+
|   Buffer Pool       |   <- fixed-size area of RAM holding cached disk pages
| [pg7][pg3][pg91]... |
+-------------------+
        |  (miss: page not cached)          (hit: page already in RAM, no disk I/O)
        v
     Disk (500GB)
```

## Problem
When a query needs a page that isn't currently cached (a "buffer miss") and the buffer pool is full, what must the buffer manager do, and why does this decision matter for performance?

## Solution
The buffer manager must **evict** some currently-cached page to make room for the new one, using a replacement policy (commonly LRU or a variant like clock/second-chance). If it evicts a page that a query needs again very soon, that query now has to pay for a disk read again ("buffer thrashing") - so the quality of the eviction policy directly determines your effective cache hit rate, and hit rate is often the single biggest factor in real-world query latency, since RAM access is orders of magnitude faster than disk.

## Takeaway
The buffer manager is the layer between your queries and physical disk I/O - understanding your buffer pool's hit ratio (and sizing it appropriately relative to your "hot" working set) is usually a bigger performance lever than most query-level tuning.
