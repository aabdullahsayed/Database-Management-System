# Wound Wait

## Scenario
Your team is comparing Wound-Wait against Wait-Die for a high-contention system, and wants to understand the practical difference: which scheme actively kills the *other* transaction instead of aborting itself.

## Problem
Transaction T1 (older) requests a lock held by T2 (younger). What does Wound-Wait dictate here, versus the reversed case (T2, younger, requests a lock held by T1, older)?

## Solution
Wound-Wait rule: when Ti requests a lock held by Tj,
- If Ti is **older** than Tj: Ti **wounds** Tj (forcibly aborts the younger transaction Tj, and Ti takes the lock).
- If Ti is **younger** than Tj: Ti **waits** (a younger transaction politely waits for the older one to finish).

So: T1 (older) requests a lock held by T2 (younger) -> **T2 is wounded (aborted)**, T1 proceeds immediately.
Reversed: T2 (younger) requests a lock held by T1 (older) -> T2 **waits**.

Contrast with Wait-Die: in Wait-Die, the *requesting* transaction is the one that dies if it's younger; in Wound-Wait, the *requesting* transaction (if older) preemptively kills the *holder*. Both prevent deadlock cycles, but Wound-Wait tends to cause fewer restarts in practice for long-running older transactions, since it actively clears obstacles rather than waiting to be aborted itself.

## Takeaway
Wound-Wait and Wait-Die are mirror-image deadlock prevention schemes - both use timestamps to break potential cycles, but Wound-Wait has older transactions preempt (kill) younger lock-holders, while Wait-Die has younger requesters self-abort.
