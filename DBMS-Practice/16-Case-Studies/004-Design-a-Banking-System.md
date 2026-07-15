# Design a Banking System

## Scenario
A banking backend needs accounts, transfers between accounts, and a full transaction ledger for audit/compliance - with the hard requirement that money can never be created or destroyed by a bug (the sum of all balances must always be internally consistent and traceable).

## Requirements
- Every balance change must be traceable to a specific transaction/ledger entry (no silent balance edits).
- A transfer must debit one account and credit another atomically.
- Historical ledger entries must be immutable (append-only, never UPDATE or DELETE).

## Diagram
```
Account                    LedgerEntry (append-only, immutable)
+-----------+     1     M   +----------------+
| acct_id   |------------->| entry_id        |
| balance   |               | account_id  FK  |
+-----------+               | amount (+/-)    |   <- positive=credit, negative=debit
                             | transfer_id     |   <- groups the debit+credit pair
                             | created_at      |
                             +----------------+
```

## Schema
```sql
CREATE TABLE Account (
    account_id SERIAL PRIMARY KEY,
    owner_name VARCHAR(100) NOT NULL,
    balance    DECIMAL(14,2) NOT NULL DEFAULT 0
);

CREATE TABLE LedgerEntry (
    entry_id    BIGSERIAL PRIMARY KEY,
    account_id  INT NOT NULL REFERENCES Account(account_id),
    transfer_id UUID NOT NULL,                 -- links the debit+credit pair of one transfer
    amount      DECIMAL(14,2) NOT NULL,        -- negative = debit, positive = credit
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
    -- no UPDATE/DELETE ever permitted on this table - enforce via DB permissions/trigger
);
```

## Transfer logic (application/procedure)
```sql
BEGIN;

-- debit sender
UPDATE Account SET balance = balance - 100.00
WHERE account_id = 1 AND balance >= 100.00;   -- fails (0 rows) if insufficient funds

INSERT INTO LedgerEntry (account_id, transfer_id, amount)
VALUES (1, :transfer_uuid, -100.00);

-- credit receiver
UPDATE Account SET balance = balance + 100.00 WHERE account_id = 2;

INSERT INTO LedgerEntry (account_id, transfer_id, amount)
VALUES (2, :transfer_uuid, 100.00);

COMMIT;
```

## Key design decisions
- `LedgerEntry` is append-only and immutable by policy (revoke `UPDATE`/`DELETE` privileges at the database level for the application role) - this is what makes the system auditable: `balance` is technically a cached/derivable value (`SUM(amount) WHERE account_id = X`), and the ledger is the actual source of truth.
- The `WHERE balance >= 100.00` condition on the debit UPDATE is an atomic check-and-update in one statement, preventing a race condition where two concurrent transfers both pass a separate balance check before either commits (see Concurrency-Control/Lost-Update).
- `transfer_id` links the two ledger entries of a single transfer together, so any individual transfer can be reconstructed and audited as a matched debit/credit pair - money is never "created" because every credit has a corresponding debit.
- The whole operation is one transaction (see Transactions/Transaction-Basics): partial application (debit succeeds, credit fails) would literally destroy money, so atomicity here isn't optional.

## Takeaway
Financial systems are the canonical use case for treating the transaction log/ledger as the actual source of truth (event-sourcing style) rather than trusting a single mutable `balance` column - it gives you both a stronger correctness guarantee and a built-in audit trail.
