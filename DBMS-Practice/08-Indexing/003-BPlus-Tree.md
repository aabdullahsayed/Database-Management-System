# B+ Tree

## Scenario
Your DBA mentions "Postgres uses a B+Tree, not a plain B-Tree" for its default index, and you want to understand the practical difference for range queries.

## Diagram
```
B-Tree: data stored in internal AND leaf nodes
B+Tree: data (or row pointers) stored ONLY in leaf nodes; leaves linked in a list

B+Tree leaves:  [10|20] -> [30|40] -> [50|60] -> [70|80]
                  (linked list across all leaf nodes, left to right)
```

## Problem
Why does a B+Tree's leaf-linking make `WHERE price BETWEEN 30 AND 70` faster than a plain B-Tree would be?

## Solution
In a plain B-Tree, data can live in internal nodes too, so a range scan may require repeated tree traversals (up and down) to collect all values in range. In a B+Tree, **all** actual data/row-pointers live only in leaf nodes, and those leaves are linked together in sorted order. Once you find the starting leaf (`30`), you just walk the linked list forward (`30 -> 40 -> 50 -> 60 -> 70`) without re-traversing the tree - a simple, fast sequential scan across leaves.

## Takeaway
B+Trees optimize specifically for range scans by linking leaf nodes into a sorted chain - this is why they're the standard choice for relational database indexes, where range queries (`BETWEEN`, `<`, `>`, `ORDER BY`) are extremely common.
