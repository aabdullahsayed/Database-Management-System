# Participation

## Scenario
Business rule: "Every order must have at least one order item, but a product does not need to appear in any order to exist in the catalog."

## Diagram
```
                    total participation (double line)
Order  ||===========<  OrderItem  >===========|| Product
        (every Order must have >=1 OrderItem)   (partial: a Product
                                                   may appear in 0 orders)
```

## Problem
Translate this participation constraint into something enforceable in the schema, and note what plain SQL foreign keys can and cannot enforce here.

## Solution
`OrderItem.order_id` and `OrderItem.product_id` as `NOT NULL` foreign keys enforce that every *order item* references a valid order and product (partial participation of Order and Product in the OrderItem relationship is trivially satisfiable).

But "every Order must have at least one OrderItem" (total participation of Order) is **not** directly expressible as a simple foreign key constraint, because that constraint lives on the child table, not the parent. Plain FKs can't force a parent row to have at least one child. You typically enforce this at the application/transaction level (create the order and its first item in the same transaction) or with a deferred trigger/constraint.

## Takeaway
Total participation constraints ("must have at least one related row") are a known blind spot of plain foreign keys - they need application-level transactions, triggers, or check constraints with subqueries to enforce properly.
