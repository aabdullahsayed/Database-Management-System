# File Organization

## Scenario
You're choosing how a new time-series `metrics` table (append-heavy, mostly queried by time range, rarely updated) should physically store its rows on disk, versus a `users` table (frequent single-row lookups and updates by id).

## Problem
Which file organization strategy fits each table's access pattern better - heap, sequential/sorted, or hash?

## Solution
`metrics` (append-heavy, range queries by time): a heap organization with data naturally appended in time order (or explicitly clustered/partitioned by time) works well, since inserts are cheap appends and range scans read mostly-sequential disk blocks.

`users` (frequent point lookups/updates by id): benefits from being organized (clustered) by `id`, so single-row lookups and updates by primary key are fast and don't require scanning unrelated data.

## Takeaway
File organization should match the dominant access pattern - append-heavy + range-scan workloads favor sequential/time-ordered layouts; point-lookup-heavy workloads favor clustering by the lookup key. Choosing wrong forces the database (or you) to compensate with more indexes and I/O later.
