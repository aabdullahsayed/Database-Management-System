# Heap File

## Scenario
A new `activity_log` table just appends new rows with no particular order requirement - you want inserts to be as cheap as possible, and don't care about physical ordering.

## Diagram
```
Heap file: rows placed wherever there's free space, no guaranteed order
[row A][row C][row B][row D][free space][row E]...
   insert = find/allocate free space, write, done (fast, O(1)-ish)
   find a specific row = must scan (or rely on a separate index) - no ordering to exploit
```

## Problem
Why is a heap organization a good default for `activity_log`, and what's the tradeoff?

## Solution
In a heap file, new rows are simply placed in the next available free space - insertion is fast and doesn't require maintaining any sort order. This fits `activity_log`, which is write-heavy and rarely needs range-ordered physical scanning (queries usually go through a separate index on `created_at` or `user_id` rather than relying on physical order).

**Tradeoff**: without an index, finding specific rows requires a full scan since there's no inherent ordering to binary-search or seek within - which is why heap tables are almost always paired with one or more secondary indexes for anything beyond "scan everything."

## Takeaway
Heap file organization optimizes for fast, unordered inserts at the cost of needing indexes for any targeted lookup - it's the right default for high-volume, append-only tables like logs and event streams.
