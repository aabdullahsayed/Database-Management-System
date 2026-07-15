# Stored Procedures

## Scenario
A "transfer funds between accounts" operation must debit one account and credit another atomically, and you want this logic to live in the database itself so it can't be bypassed or duplicated incorrectly by different application services.

## Solution
```sql
CREATE PROCEDURE transfer_funds(
    sender_id INT, receiver_id INT, amount DECIMAL(10,2)
)
LANGUAGE plpgsql
AS $$
BEGIN
    IF (SELECT balance FROM accounts WHERE id = sender_id) < amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;

    UPDATE accounts SET balance = balance - amount WHERE id = sender_id;
    UPDATE accounts SET balance = balance + amount WHERE id = receiver_id;
END;
$$;

-- usage:
CALL transfer_funds(101, 202, 50.00);
```

Because both updates happen inside the procedure as one unit, and Postgres procedures run within a transaction context, this guards against a partial transfer (money leaving one account but never arriving at the other) far more reliably than trusting every calling service to wrap the two updates in a transaction correctly.

## Takeaway
Stored procedures push critical, must-be-atomic business logic into the database itself, so it's enforced once, consistently, regardless of which application or language calls it - useful for financial operations and other invariant-critical logic.
