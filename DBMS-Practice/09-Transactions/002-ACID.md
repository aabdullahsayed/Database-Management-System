# ACID

## Scenario
Your lead asks you to explain, in terms of a concrete bug each one prevents, what ACID actually buys you - not just the acronym.

## Solution
- **Atomicity**: the checkout transaction above - all 3 statements succeed or none do. Prevents: charging a customer without creating their order.
- **Consistency**: constraints (e.g. `CHECK (quantity >= 0)`) are never violated, even mid-transaction. Prevents: inventory going negative because two transactions raced past a check.
- **Isolation**: concurrent transactions don't see each other's uncommitted changes. Prevents: transaction B reading a half-updated balance from transaction A that hasn't committed yet (dirty read).
- **Durability**: once `COMMIT` returns success, the change survives a crash immediately after. Prevents: "the payment succeeded, but the server crashed before writing to disk, and the record vanished after reboot."

## Takeaway
ACID isn't abstract theory - each letter maps directly to a specific class of real production bug (partial writes, invalid states, race conditions, data loss on crash) that you'd otherwise have to solve yourself in application code, badly.
