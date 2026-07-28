# 9. Concurrency Control & Locking

## Quick Refresher
- **Lock**: a mechanism preventing conflicting access to the same data by multiple transactions.
- **Shared (S) lock**: multiple transactions can hold it simultaneously — used for reads.
- **Exclusive (X) lock**: only one transaction can hold it — used for writes.
- **Deadlock**: two transactions each hold a lock the other needs, waiting forever.
- **Two-Phase Locking (2PL)**: a transaction acquires all locks it needs first (growing phase), then releases them (shrinking phase) — guarantees serializability.

## Practice Problems

### Q1 (Basic). What's the difference between a shared lock and an exclusive lock?
**Answer:** A shared (S) lock allows multiple transactions to read the same data simultaneously but blocks any writes. An exclusive (X) lock is needed to write, and blocks all other reads and writes on that data until released.

### Q2 (Basic). What is a deadlock, and how does a database typically resolve one?
**Answer:** A deadlock is a cycle of transactions each waiting on a lock held by the next. Databases run a **deadlock detection** algorithm (looking for cycles in a "waits-for" graph) and resolve it by picking a "victim" transaction to **abort and roll back**, letting the others proceed.

### Q3 (Intermediate). What does Two-Phase Locking (2PL) guarantee, and what's the trade-off?
**Answer:** 2PL guarantees **serializability** — the outcome of concurrently running transactions is equivalent to running them one at a time in some order. The trade-off is reduced concurrency/throughput, since transactions hold locks longer (they can't release any lock until they're done acquiring all locks), increasing the chance of blocking and potential deadlocks.

### Q4 (Intermediate). Explain the difference between "Strict 2PL" and plain 2PL.
**Answer:** In plain 2PL, locks can be released as soon as the shrinking phase begins (even before commit). In **Strict 2PL**, all exclusive locks are held until the transaction actually **commits or rolls back** — this prevents other transactions from reading uncommitted (dirty) data, which is why almost all real databases use strict 2PL in practice.

### Q5 (Intermediate). What is "lock escalation," and why might a database do it?
**Answer:** Lock escalation is when the database converts many fine-grained locks (e.g., thousands of row-level locks) into one coarser lock (e.g., a full table lock) to reduce the memory/overhead of tracking so many individual locks — at the cost of reduced concurrency, since the entire table becomes temporarily unavailable to other writers.

### Q6 (Advanced/Interview). Given two transactions:
```
T1: locks Row A, then wants to lock Row B
T2: locks Row B, then wants to lock Row A
```
**Explain the deadlock that occurs, and describe two different strategies to prevent it (not just detect it).**
**Answer:** T1 holds A and waits for B; T2 holds B and waits for A — a circular wait, deadlock.
**Prevention strategies:**
1. **Lock ordering**: always acquire locks in a fixed global order (e.g., always lock the row with the smaller primary key first) — this breaks the possibility of a circular wait entirely.
2. **Timeouts**: if a transaction waits too long for a lock, abort it automatically and retry — doesn't prevent deadlock from forming but limits how long the system stays stuck.
3. (Bonus) **Wait-Die / Wound-Wait schemes**: use transaction timestamps to decide whether an older transaction should wait or force a younger one to abort, avoiding indefinite cycles.

### Q7 (Advanced/Interview). Why can `SELECT ... FOR UPDATE` be a double-edged sword in a high-throughput system? Give a concrete scenario.
**Answer:** `SELECT ... FOR UPDATE` takes an exclusive lock on the selected rows immediately, blocking other transactions from reading (in some isolation levels) or writing them until commit. In a high-throughput checkout flow (e.g., many users trying to buy the last few units of a popular product), if every request locks the same inventory row and only releases it at the end of a slow multi-step transaction (payment processing, etc.), requests queue up behind that lock, tanking throughput and increasing timeout errors — even though the actual conflict rate might be low. Mitigation: keep the locked transaction as short as possible (lock, decrement, commit immediately; do slow work like payment processing **outside** the lock), or use optimistic concurrency control instead if contention is rare.
