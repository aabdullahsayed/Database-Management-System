# Tables and Relations

## Scenario
You're modeling a "relation" (table) for orders in an online store. A junior dev insists on calling it an "array of order objects." Your tech lead pushes back: a relational table is a *set* of tuples, not a list.

## Diagram
```
orders
+---------+------------+--------+
| order_id| customer_id| total  |
+---------+------------+--------+
| 101     | 5          | 49.99  |
| 102     | 7          | 12.50  |
+---------+------------+--------+
```

## Problem
Why does it matter that a relation is a *set* of tuples (no duplicates, no defined order) rather than a list/array? Give one concrete bug this distinction prevents.

## Solution
If rows had a guaranteed order and duplicates were fine, application code might rely on "the 3rd row inserted is always at position 3" - but a DBMS is free to store/return rows in any physical order for performance (e.g. after a reorg or index rebuild). Relying on implicit order without an `ORDER BY` is a real, common bug: a report that "used to" come back in insertion order silently reorders after a schema change or vacuum, breaking pagination or displayed rankings.

## Takeaway
Never assume row order without an explicit `ORDER BY`. A relation is a set of tuples; order and duplication are not guaranteed unless you ask for them.
