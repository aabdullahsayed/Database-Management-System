# Hash File

## Scenario
A `sessions` table is looked up ONLY by exact `session_token` (never by range, never sorted) - millions of times per second, always an exact-match lookup.

## Diagram
```
Hash file:
hash("abc123...") -> bucket 7  -> [row for session abc123...]
hash("xyz789...") -> bucket 2  -> [row for session xyz789...]

Lookup: hash the key -> jump directly to the bucket -> O(1) average
```

## Problem
Why is a hash-organized structure (or hash index) a good fit here, and what would make it a bad fit if requirements changed?

## Solution
Since every access pattern is "give me the exact row for this exact token," hashing the key to directly locate its storage bucket gives O(1) average-case lookup - no need to traverse a sorted tree structure at all, which would be needlessly slower (O(log n)) for a pure equality-lookup workload.

**Bad fit if requirements change**: if product later asks for "show me all sessions created in the last hour" (a range query on `created_at`), a hash structure on `session_token` is useless for that - hashing destroys ordering entirely. You'd need a separate B-Tree index on `created_at` to support that query pattern.

## Takeaway
Hash-based storage/indexing is the right specialized tool exclusively for pure equality lookups; the moment any range, sort, or prefix query enters the picture, you need a tree-based (B+Tree) structure instead.
