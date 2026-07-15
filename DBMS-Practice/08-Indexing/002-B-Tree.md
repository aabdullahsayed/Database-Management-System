# B-Tree

## Scenario
You want to understand why your database chose a B-Tree structure for its default index type, rather than, say, a simple hash table.

## Diagram
```
                [ 50 ]
              /        \
        [20, 35]       [70, 90]
       /   |   \       /   |   \
   [..] [..] [..]   [..] [..] [..]     <- leaf nodes, sorted, often linked
```

## Problem
A hash index can do O(1) equality lookups (`WHERE id = 5`), which sounds faster than a B-Tree's O(log n). Why do relational databases still default to B-Trees for general-purpose indexes?

## Solution
Hash indexes only support exact-match equality (`=`). They cannot efficiently answer range queries (`WHERE price BETWEEN 10 AND 50`), prefix/ordering queries (`ORDER BY created_at`), or `<`/`>` comparisons, because hashing destroys the original ordering of the data.

A B-Tree keeps keys **sorted**, and its leaf nodes are typically linked in order, so it efficiently supports equality *and* range scans *and* ordered traversal - covering the vast majority of real-world query patterns with one structure, at the cost of O(log n) instead of O(1) for pure equality lookups.

## Takeaway
B-Trees are the default index structure because they support the widest range of query patterns (equality, range, ordering) reasonably efficiently; hash indexes are a narrower, faster tool reserved for pure equality lookups only.
