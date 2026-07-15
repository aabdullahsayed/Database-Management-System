# Lossless Decomposition

## Scenario
A teammate decomposes `R(A, B, C)` into `R1(A, B)` and `R2(B, C)` for a "cleaner" schema. You suspect this loses information and want to prove it with a test query.

## Problem
Given FD `B -> C` only (no FD from B to A, or A to B), is `R1(A,B) JOIN R2(B,C)` guaranteed to reconstruct R exactly (lossless)?

## Solution
The lossless-join test: a decomposition of R into R1 and R2 is lossless if and only if
`(R1 ∩ R2) -> R1` or `(R1 ∩ R2) -> R2` holds.

Here `R1 ∩ R2 = {B}`. Check: does `B -> R1` (i.e. `B -> A`)? No FD says that. Does `B -> R2` (i.e. `B -> C`)? **Yes**, we're given `B -> C`, and `B` trivially determines itself, so `B -> {B,C}` = `B -> R2` holds.

So this decomposition **is lossless** - because the common attribute `B` is a determinant of the full `R2` side.

**Contrast:** if instead the only given FD were `A -> B` (nothing involving C), then `B -> R1` and `B -> R2` would both fail, and joining `R1(A,B)` with `R2(B,C)` on `B` could produce spurious extra rows not in the original R - a lossy decomposition.

## Takeaway
Always run the lossless-join test (`R1 ∩ R2` must functionally determine at least one of the two sides) before shipping a decomposition - "it looks cleaner" is not proof it preserves your data.
