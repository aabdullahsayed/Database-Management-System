# Find Candidate Key

## Scenario
A migration script needs a primary key for a legacy table, but nobody documented one. You're given the FDs from a data audit.

## Problem
Given `R(A, B, C, D, E)` with FDs:
```
A -> B
B -> C
A -> D
D -> E
```
Find a candidate key for R.

## Solution
Try `A` alone: compute `A+`.
- `A -> B`: add B. `{A,B}`
- `B -> C`: add C. `{A,B,C}`
- `A -> D`: add D. `{A,B,C,D}`
- `D -> E`: add E. `{A,B,C,D,E}`

`A+` = all attributes of R, so **A is a superkey**. Since it's a single attribute, it's automatically minimal (no smaller nonempty subset exists) -> **A is a candidate key**.

Check no other single attribute works: `B+ = {B,C}` (missing A,D,E) - not a key. Same logic rules out C, D, E alone.

**Candidate key: {A}**

## Takeaway
To find candidate keys, compute closures of attribute subsets (start small - single attributes - before trying combinations) and check which minimal sets reach all attributes of R.
