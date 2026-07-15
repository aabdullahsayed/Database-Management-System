# Rollback

## Scenario
Mid-transaction, your application code catches an exception (e.g. the payment gateway times out) after already having run the `INSERT INTO orders` and `UPDATE inventory` statements. You must undo those partial changes.

## Solution
```sql
BEGIN;

INSERT INTO orders (customer_id, total) VALUES (5, 49.99);
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 100;

-- payment gateway call fails / times out in application code
ROLLBACK;
-- both the INSERT and the UPDATE above are undone, as if they never happened
```

`ROLLBACK` discards every change made since `BEGIN`, returning the database to the state it was in before the transaction started. In application code, this typically means wrapping the transaction in a try/catch and calling `ROLLBACK` in the catch/finally block whenever any step fails.

## Takeaway
Never leave a transaction open on error without an explicit `ROLLBACK` - an uncommitted transaction that's neither committed nor rolled back can hold locks indefinitely, and your application must always have an error path that rolls back.
