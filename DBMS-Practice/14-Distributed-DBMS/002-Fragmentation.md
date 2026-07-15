# Fragmentation

## Scenario
Your `orders` table has grown to 2 billion rows and won't fit comfortably on one machine. You need to split it across multiple database nodes.

## Diagram
```
Horizontal fragmentation (sharding):        Vertical fragmentation:
Node 1: orders WHERE region='US'            Node 1: orders(id, customer_id, total)
Node 2: orders WHERE region='EU'            Node 2: orders(id, shipping_address, notes)
Node 3: orders WHERE region='ASIA'            (split by COLUMN, same rows split across
  (split by ROW, same columns everywhere)      two column-groups on different nodes)
```

## Problem
For `orders`, which fragmentation strategy fits better, given that most queries filter by `region` (e.g. regional compliance/reporting) but need all columns of a matched row?

## Solution
**Horizontal fragmentation (sharding by region)** fits better here: since queries already filter by `region`, each query can be routed to exactly the node holding that region's rows, and every relevant column is available locally on that node - no cross-node joins needed to reconstruct a row. Vertical fragmentation would instead force every query needing `shipping_address` to talk to a different node than one needing `total`, adding unnecessary cross-node round trips even for queries that only touch one region.

## Takeaway
Choose horizontal fragmentation when your query patterns naturally partition by a key (e.g. region, tenant_id, customer_id) and you usually need whole rows; choose vertical fragmentation only when distinct groups of columns are consistently accessed together but rarely together with other column groups.
