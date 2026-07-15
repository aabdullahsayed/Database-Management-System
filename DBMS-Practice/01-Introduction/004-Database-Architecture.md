# Database Architecture

## Scenario
Your app is slow under load. You want to understand where a query actually goes between your app code and the disk, so you can figure out where to optimize.

## Diagram
```
 Application
     |
     v
 +-------------------+
 | Query Processor    |  <- parses SQL, builds a plan
 +-------------------+
     |
     v
 +-------------------+
 | Optimizer           |  <- picks the cheapest execution plan
 +-------------------+
     |
     v
 +-------------------+
 | Execution Engine    |  <- runs the plan (scans, joins, sorts)
 +-------------------+
     |
     v
 +-------------------+
 | Storage / Buffer     |  <- pages cached in memory
 | Manager (Buffer Pool) |
 +-------------------+
     |
     v
 +-------------------+
 |     Disk (files)     |
 +-------------------+
```

## Problem
A query that used to run in 5ms now takes 800ms after the table grew to 50 million rows. Using the diagram above, list two layers where the slowdown could originate and how you'd check each.

## Solution
- Optimizer layer: the query plan may have flipped from an index scan to a full table scan once the table grew (stats became stale, or the optimizer decided the index was no longer selective enough). Check with `EXPLAIN ANALYZE`.
- Storage/Buffer layer: at 50M rows, the working set of pages needed may no longer fit in the buffer pool (RAM cache), causing many more disk reads. Check buffer pool hit ratio / cache stats.

## Takeaway
A DBMS is a layered pipeline (parse -> optimize -> execute -> fetch from storage). Debugging slow queries means figuring out which layer regressed, not just "the database is slow."
