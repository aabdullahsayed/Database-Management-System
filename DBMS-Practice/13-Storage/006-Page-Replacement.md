# Page Replacement

## Scenario
Your database's buffer pool is full and needs to evict a page to make room for a newly requested one. You're comparing two candidate replacement policies: strict LRU versus Clock (second-chance).

## Diagram
```
LRU (strict):                          Clock / Second-Chance:
[most recent .......... least recent]  circular buffer with a reference bit per page
  evict from the "least recent" end      sweep pointer: if ref_bit=1, clear it & skip (give
  (requires tracking exact recency        it a "second chance"); if ref_bit=0, evict it
   order - more bookkeeping overhead)     (approximates LRU cheaply, less bookkeeping)
```

## Problem
Why might a database prefer the Clock algorithm over strict LRU, given LRU is theoretically "more accurate"?

## Solution
Strict LRU requires updating a precise recency ordering (e.g. a doubly linked list) on every single page access, which under high concurrency means constant contention on that shared ordering structure - a real bottleneck at scale. Clock/second-chance approximates LRU behavior using a single reference bit per page (cheap to set/check) and a sweeping pointer, achieving similar eviction quality in practice with far less synchronization overhead - a classic "good enough and much cheaper" engineering tradeoff.

## Takeaway
The "theoretically best" algorithm (strict LRU) isn't always the practical choice under high concurrency - approximation algorithms like Clock trade a small amount of eviction accuracy for a large reduction in synchronization overhead, which is why most real database buffer managers use Clock or similar approximations rather than textbook LRU.
