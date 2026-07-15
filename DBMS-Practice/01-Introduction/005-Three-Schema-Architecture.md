# Three-Schema Architecture

## Scenario
Your backend team wants to migrate the `users` table's physical storage - moving from row-oriented storage to a partitioned layout by `signup_date` - to speed up analytics queries, without changing a single line of application code or breaking the `SELECT * FROM users WHERE id = ?` queries the API layer already runs.

## Diagram
```
External Level     App A view: (id, name, email)      App B view: (id, email, plan)
                              \                              /
                               \                            /
Conceptual Level                    users(id, name, email, plan, signup_date)
                                                |
Internal Level                 Physical storage: partitioned by signup_date,
                                B+Tree index on id, compressed columns
```

## Problem
Explain why this migration is safe under the three-schema architecture, and name which schema level changed.

## Solution
Only the **internal (physical) schema** changed - how rows are laid out on disk (partitioning by `signup_date`, index structures). The **conceptual schema** (the logical table `users(id, name, email, plan, signup_date)`) stayed the same, and so did the **external schemas** (each application's view/queries). Because the DBMS provides physical data independence, application code querying by `id` never needs to know or care how rows are physically partitioned.

## Takeaway
The three-schema architecture (external / conceptual / internal) exists specifically so that physical storage changes (like this migration) don't ripple up and break application code - that separation is called data independence.
