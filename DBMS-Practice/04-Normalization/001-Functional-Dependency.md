# Functional Dependency

## Scenario
You inherit an `orders` table that mixes order and customer data in one wide table, and a bug where updating a customer's address in one order row doesn't update it in their other orders.

## Diagram
```
orders(order_id, customer_id, customer_address, product_id, product_price)

customer_id -> customer_address     (FD: customer_id determines address)
product_id  -> product_price        (FD: product_id determines price)
```

## Problem
Identify the functional dependencies (FDs) hiding in this table and explain how they caused the address bug.

## Solution
A functional dependency `X -> Y` means: for any two rows with the same X, Y must also be the same. Here, `customer_id -> customer_address` should hold, but because `customer_address` is duplicated across every order row for that customer, updating it in one row (e.g. order 101) without updating all others violates the FD - now two rows with the same `customer_id` disagree on `customer_address`. That inconsistency is the bug.

## Takeaway
Anomalies like this are always a symptom of an FD that isn't respected by the table's key structure - this is exactly what normalization (2NF/3NF) is designed to fix, by separating `customer_id -> customer_address` into its own `Customer` table.
