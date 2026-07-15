# Query Optimization

## Scenario
You write `SELECT * FROM orders o JOIN customers c ON o.customer_id = c.id WHERE c.country = 'US'`. Logically this could be executed as "join everything, then filter by country" or "filter customers by country first, then join" - both produce the identical result set.

## Diagram
```
Naive plan:                          Optimized plan:
   Filter(country='US')                 Join
        |                                 / \
      Join                       Filter(country='US')  orders
       /  \                          |
  orders  customers               customers
  (join ALL rows first,          (filter customers down to 'US' FIRST,
   then throw most away)          then join only the smaller matching set)
```

## Problem
Both plans are logically equivalent (same result). Why does the optimizer prefer pushing the filter down before the join?

## Solution
This is called **predicate pushdown** - moving a `WHERE` filter as early as possible in the execution plan, ideally before an expensive join. If only 2% of customers are in the US, filtering first shrinks the customers side of the join to 2% of its original size *before* the expensive join operation runs, instead of joining the full table and discarding 98% of the result afterward. Same output, dramatically less work.

## Takeaway
The query optimizer's job is to find a plan that's logically equivalent to what you wrote but cheaper to execute - predicate pushdown, join reordering, and choosing index vs. scan access paths are all examples of transformations the optimizer applies automatically, which is why `EXPLAIN` output often looks structurally different from your literal SQL.
