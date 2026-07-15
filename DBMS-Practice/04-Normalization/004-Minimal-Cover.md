# Minimal Cover

## Scenario
An auto-generated FD list from a schema analysis tool gives you redundant, bloated dependencies. Before you use them to normalize a table, you need to simplify (minimize) the set.

## Problem
Given FDs on `R(A, B, C)`:
```
A -> B
A -> BC
B -> C
AB -> C
```
Find the minimal cover.

## Solution
1. **Split right-hand sides** to single attributes:
   `A -> B`, `A -> C`, `B -> C`, `AB -> C`
2. **Remove redundant FDs**: check if `A -> C` is implied by the others without it.
   Using `A -> B` and `B -> C`: `A -> B -> C`, so `A -> C` is redundant. Remove it.
   Remaining: `A -> B`, `B -> C`, `AB -> C`
3. **Remove extraneous attributes on the left**: for `AB -> C`, check if `B -> C` alone already gives this (yes, since B is in AB and B alone -> C). So `AB -> C` is redundant given `B -> C`. Remove it.
   Remaining: `A -> B`, `B -> C`

**Minimal cover: { A -> B, B -> C }**

## Takeaway
A minimal cover has three properties: every right-hand side is a single attribute, no FD is redundant (removable without losing closure equivalence), and no left-hand side has extraneous attributes. Always minimize before using FDs to design normalized tables - it directly determines your 3NF decomposition.
