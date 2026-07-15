# Graph Database

## Scenario
"People you may know" needs to find friends-of-friends-of-friends (3 hops) who aren't already your friend, across a social network with 50 million users and billions of friendship edges - and it needs to respond in under 100ms.

## Diagram
```
Relational approach (self-joins):            Graph approach (native traversal):
friends JOIN friends JOIN friends              (You)-[FRIEND]->(A)-[FRIEND]->(B)-[FRIEND]->(C)
  -> 3-way self join on a huge table            traverse edges directly, no join cost -
  -> cost grows very fast with each hop          each hop follows a pointer/index locally
```

## Problem
Why does a relational multi-way self-join struggle here specifically, while a graph database handles it comfortably?

## Solution
Each additional "hop" in a relational self-join multiplies the join's intermediate result size and cost - a 3-hop friend-of-friend-of-friend query on billions of edges can produce an enormous intermediate row count before final filtering, even with indexes on both sides of the join. A graph database stores each node's adjacent edges directly (like an adjacency list optimized for traversal), so "follow this node's friend edges" is a fast, local, near-constant-time operation regardless of overall graph size - the cost scales with the number of hops and the fan-out at each hop, not with the total size of the dataset.

## Takeaway
When your dominant query pattern is "traverse relationships N hops deep," a graph database's native edge-traversal will outperform relational self-joins by a wide and growing margin as hop count increases - this is the textbook use case graph databases are built for.
