# Sequential File

## Scenario
A nightly batch report needs to scan an entire `daily_snapshots` table in date order, and the table is rarely written to outside of the nightly batch job itself - a case where physically sorted storage pays off.

## Diagram
```
Sequential (sorted) file, physically ordered by date:
[2026-07-01][2026-07-02][2026-07-03][2026-07-04]...
   range scan "give me July 1 through July 10" = read one contiguous disk region
   insert a NEW row for 2026-07-01.5 = must shift/insert in the middle (expensive)
```

## Problem
Why does sorted physical storage help the nightly report, and why would this same layout be a poor choice for a table with frequent, scattered inserts?

## Solution
Because the file is physically sorted by date, a range query like "July 1 to July 10" reads one contiguous block of disk - very fast, minimal seek overhead, and naturally supports binary search for a starting point. This is ideal for `daily_snapshots`, which is written once per day (append at the end, which stays sorted) and read in large date ranges.

But if inserts happened at arbitrary, out-of-order dates constantly, maintaining strict sorted order would require shifting existing data to make room for each new row - expensive at scale. This is exactly why B+Tree indexes exist: they give you "logically sorted, fast range scan" behavior without requiring the underlying physical file itself to stay perfectly sorted on every insert.

## Takeaway
Truly sequential (physically sorted) files are great for scan-heavy, append-at-the-end, rarely-updated-out-of-order data; for anything with scattered inserts, a B+Tree index gives you the range-scan benefits without the insert cost of physically re-sorting the file.
