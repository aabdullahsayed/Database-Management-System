# Transaction Basics

## Scenario
Your checkout flow does three separate operations: create order, deduct inventory, charge payment. Currently each is a separate, independent database call - if the server crashes between step 2 and step 3, inventory is deducted but no order/payment exists.

## Solution
```sql
BEGIN;

INSERT INTO orders (customer_id, total) VALUES (5, 49.99) RETURNING id;
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 100;
INSERT INTO payments (order_id, amount, status) VALUES (:order_id, 49.99, 'charged');

COMMIT;
```

Wrapping all three statements in `BEGIN ... COMMIT` groups them into a single transaction: either all three take effect, or (if anything fails and you `ROLLBACK`, or the connection drops before `COMMIT`) none of them do. Half-completed states become impossible.

## Takeaway
A transaction is the unit of "all-or-nothing" - any sequence of operations that must succeed or fail together (especially anything touching money or inventory) belongs inside `BEGIN`/`COMMIT`, never as separate unguarded statements.
