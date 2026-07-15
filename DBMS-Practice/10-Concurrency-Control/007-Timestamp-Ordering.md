# Timestamp Ordering

## Scenario
Instead of locks, your database (or a distributed system you're designing) assigns each transaction a unique timestamp when it starts, and uses it to decide the order transactions are allowed to "appear" to have run in, without ever blocking on locks.

## Problem
Given transactions T1 (timestamp 10) and T2 (timestamp 20), T2 tries to write a value that T1 (the "older" transaction, lower timestamp) has already read. What should happen, and why?

## Solution
Timestamp ordering assigns each transaction a fixed timestamp and enforces: a transaction may only read/write data in an order consistent with timestamp order - a transaction is not allowed to write a value that a transaction with a *later* timestamp has already read (this would violate the illusion that transactions executed in timestamp order).

Here, T1 (ts=10) already read the value; T2 (ts=20) attempting to write it now would be fine (T2 is "later," writing after an earlier read is consistent with T1-then-T2 ordering) - but if the situation were reversed (an *older* transaction trying to write after a *younger* one already read the value), the older transaction would be **aborted and restarted** with a new, later timestamp, since allowing that write would violate ordering.

## Takeaway
Timestamp ordering is a lock-free (optimistic-ish) alternative to 2PL: instead of blocking transactions with locks, it detects ordering violations and aborts/restarts the offending transaction - useful in distributed systems where locking across nodes is expensive.
