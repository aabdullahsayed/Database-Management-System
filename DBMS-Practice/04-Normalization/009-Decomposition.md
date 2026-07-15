# Decomposition

## Scenario
You need to split a bloated `R(order_id, customer_id, customer_name, product_id, product_name, quantity)` table into normalized pieces, but your lead warns: "make sure the decomposition doesn't lose information or break the FDs."

## Problem
Decompose R given FDs: `order_id -> customer_id`, `customer_id -> customer_name`, `product_id -> product_name`, `{order_id, product_id} -> quantity`. What two properties must the decomposition satisfy?

## Solution
The decomposition:
```
Orders(order_id, customer_id)
Customers(customer_id, customer_name)
Products(product_id, product_name)
OrderItems(order_id, product_id, quantity)
```

Must satisfy:
1. **Lossless join**: joining the decomposed tables back together must reproduce exactly the original rows of R - no spurious rows, no lost rows.
2. **Dependency preservation**: every original FD should be checkable using only the FDs local to one of the new tables, without needing an expensive join to verify it.

Here both hold: `order_id -> customer_id` is enforceable within `Orders`, `customer_id -> customer_name` within `Customers`, etc.

## Takeaway
Decomposition isn't just "split the table" - a bad split can silently lose data (failing lossless join) or make you unable to enforce a constraint without an expensive join (failing dependency preservation). Always verify both properties.
