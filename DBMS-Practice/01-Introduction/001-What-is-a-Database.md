# What is a Database

## Scenario
You are building a to-do app. Day 1, you store tasks in a JSON file on disk. It works for you alone. Three weeks later your manager asks: "Can two people use it at once? Can we search 2 million tasks in under 100ms? What happens if the server crashes mid-write?"

Your JSON file cannot answer any of these questions safely.

## Problem
List what breaks in the JSON-file approach when you add:
1. Concurrent writers (two people editing at once)
2. A crash during a write
3. A query like "all tasks due this week, sorted by priority"

## Solution
1. Concurrent writers: two processes reading-modifying-writing the same file can overwrite each other's changes (lost update). A file has no locking or isolation guarantees by default.
2. Crash mid-write: if the process dies after writing half the JSON, the file is corrupted and unreadable by any reader (no atomicity, no durability guarantee).
3. Ad-hoc queries: you must load the entire file into memory and scan it in application code every time. No indexes, no query planner, O(n) scans that get worse as data grows.

A DBMS solves all three with transactions (atomicity + isolation), write-ahead logging (durability/crash recovery), and indexes + a query engine (fast lookups).

## Takeaway
A database is not "a fancy file" - it is a system that guarantees correctness (ACID) and gives you fast, declarative access to data at scale, which hand-rolled file storage cannot do reliably.
