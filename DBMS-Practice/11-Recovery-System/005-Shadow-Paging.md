# Shadow Paging

## Scenario
Your DBA mentions an older/alternative recovery technique to logging: shadow paging, used by some systems (and conceptually similar to how some copy-on-write filesystems and SQLite's rollback journal work). You want to understand the tradeoff versus WAL-based recovery.

## Diagram
```
Before update:                 During update (shadow copy):
Page Table -> [Page A]         Page Table (old) -> [Page A]        (unchanged, still valid)
                                Shadow Page Table -> [Page A']       (new, modified copy)

On COMMIT: atomically swap the "current" page table pointer to the shadow table.
On CRASH before commit: just discard the shadow pages; original Page A is untouched.
```

## Problem
Why doesn't shadow paging need undo/redo logging the way WAL-based recovery does?

## Solution
Instead of modifying data pages in place and logging the changes, shadow paging never touches the original page - it writes changes to a brand-new copy (shadow page) and only makes that copy "live" by atomically swapping a page table pointer at commit time. If a crash happens before that atomic swap, the original data was never modified at all - there's nothing to undo. If the crash happens after the swap, the new pages are already fully in place - there's nothing to redo.

**Tradeoffs vs WAL**: shadow paging avoids the complexity of undo/redo logging, but it causes data fragmentation (old pages become garbage needing collection), doesn't naturally support fine-grained concurrent transactions as well, and makes some optimizations (like grouping many small writes efficiently) harder - which is why most modern high-concurrency relational databases (Postgres, MySQL/InnoDB) use WAL-based logging instead.

## Takeaway
Shadow paging achieves atomicity via copy-on-write and an atomic pointer swap rather than logging - conceptually simpler for single-writer scenarios, but WAL-based logging has won out in most mainstream RDBMSs due to better concurrency and space efficiency.
