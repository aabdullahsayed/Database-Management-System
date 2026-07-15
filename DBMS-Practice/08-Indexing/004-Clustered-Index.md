# Clustered Index

## Scenario
You're deciding what a table's primary key should be for an `events` table that gets both heavy writes and frequent range scans by `id`. A teammate suggests a random UUID as primary key; another suggests an auto-increment integer.

## Diagram
```
Clustered index: rows are PHYSICALLY stored in key order on disk.

Auto-increment PK (sequential):        Random UUID PK:
[1][2][3][4][5][6]  -> new rows        [7f3a...][2c91...][b005...] -> new rows
   append at the end, sequential          land in RANDOM disk pages,
   disk writes (fast)                     causing page splits everywhere (slow)
```

## Problem
Why does a random UUID primary key cause worse write performance than an auto-increment integer, specifically because of clustered indexing?

## Solution
In engines like MySQL/InnoDB, the primary key IS the clustered index - rows are physically stored on disk sorted by primary key value. With a sequential auto-increment key, new rows always land at the "end" of the structure - cheap, sequential disk writes. With a random UUID, each new row's key value lands in an essentially random position among existing pages, forcing frequent **page splits** (rearranging existing data to make room) and worse cache locality, which is measurably slower at scale.

**Common compromise:** use UUIDv7 or ULIDs, which are random enough to avoid collision/guessability concerns but are time-sortable (roughly monotonic), preserving most of the sequential-insert performance benefit.

## Takeaway
The clustered index choice (usually the primary key) determines physical row layout on disk - prefer sequential or time-ordered keys for write-heavy tables; random UUIDs as clustered keys are a well-known performance anti-pattern at scale.
