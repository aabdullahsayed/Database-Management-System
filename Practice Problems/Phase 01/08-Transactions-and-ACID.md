# 8. Transactions & ACID Properties

## Quick Refresher
A **transaction** is a group of operations treated as a single unit — either all of them succeed, or none do.

- **A — Atomicity**: all-or-nothing. If any part fails, the whole transaction rolls back.
- **C — Consistency**: a transaction takes the database from one valid state to another (all constraints/rules still hold).
- **I — Isolation**: concurrent transactions don't interfere with each other's intermediate results.
- **D — Durability**: once committed, changes survive even a crash (written to persistent storage).

## Practice Problems

### Q1 (Basic). What SQL commands manage a transaction?
**Answer:** `BEGIN`/`START TRANSACTION` to start, `COMMIT` to save changes permanently, `ROLLBACK` to undo everything since the transaction started.

### Q2 (Basic). Give a real-world example of why Atomicity matters.
**Answer:** A bank transfer: subtract $100 from Account A, add $100 to Account B. If the system crashes after the subtraction but before the addition, Atomicity guarantees the whole transaction rolls back — you never end up with money "vanishing."

### Q3 (Intermediate). What is a "dirty read," and which isolation level prevents it?
**Answer:** A dirty read happens when a transaction reads data that another transaction has **modified but not yet committed** — if that other transaction rolls back, you've read data that never "really" existed. Prevented starting from the **Read Committed** isolation level (the default in most databases like PostgreSQL, Oracle, SQL Server).

### Q4 (Intermediate). List the four standard SQL isolation levels from weakest to strongest, and what anomaly each one newly prevents.
**Answer:**
1. **Read Uncommitted** — allows dirty reads (weakest).
2. **Read Committed** — prevents dirty reads (but allows non-repeatable reads).
3. **Repeatable Read** — prevents non-repeatable reads (re-reading the same row gives the same value within a transaction), but may allow phantom reads (in some DBs).
4. **Serializable** — prevents phantom reads too; transactions behave as if run one at a time (strongest, slowest).

### Q5 (Intermediate). What's the difference between a "non-repeatable read" and a "phantom read"?
**Answer:** A **non-repeatable read** happens when you read the *same row* twice in one transaction and get different values because another transaction updated it in between. A **phantom read** happens when you re-run the *same query* (e.g., "count rows where status = 'pending'") and get a **different set of rows** because another transaction inserted/deleted matching rows in between.

### Q6 (Advanced/Interview). What is the difference between optimistic and pessimistic concurrency control, and when would you choose one over the other?
**Answer:**
- **Pessimistic**: acquire a lock on data before touching it, blocking other transactions until you're done — assumes conflicts are likely. Good for high-contention writes (e.g., inventory counters during a flash sale).
- **Optimistic**: don't lock upfront; instead, check at commit time (e.g., using a version number or timestamp column) whether the data changed since you read it, and abort/retry if so — assumes conflicts are rare. Good for low-contention, read-heavy systems where locking overhead would hurt throughput unnecessarily.

### Q7 (Advanced/Interview). Explain what a "lost update" anomaly is and how you'd prevent it in application code, using an e-commerce inventory example.
**Answer:** A lost update happens when two transactions **both read** the same value (e.g., `stock = 5`), both independently decide to decrement it, and both **write back** `stock = 4` — one decrement is silently lost, even though two items were actually sold, leaving stock overcounted by 1.
**Prevention options:**
- Use `UPDATE Inventory SET stock = stock - 1 WHERE product_id = 5 AND stock > 0;` — an atomic, single-statement update where the database (not the application) reads and writes in one step, avoiding the read-then-write race entirely.
- Use **row-level locking** (`SELECT ... FOR UPDATE`) to block concurrent reads of the same row until the first transaction commits.
- Use **optimistic locking** with a version column, rejecting the second write if the version has changed since it was read.

### Q8 (Advanced/Interview). What does "Consistency" in ACID actually guarantee, and how is it different from the "C" in the CAP theorem (a very common interview trip-up)?
**Answer:** ACID's **Consistency** means a transaction only moves the database between states that satisfy all defined constraints (foreign keys, unique constraints, triggers, etc.) — it's about respecting the schema's rules. CAP's **Consistency** means every read receives the **most recent write** (or an error) across a distributed system — it's about how up-to-date/synchronized replicated copies of data are. They share a name but describe fundamentally different concerns (single-node data integrity vs. distributed-system data freshness).
