# DBMS vs File System

## Scenario
Your team stores application logs as flat `.csv` files, one per day, in `/var/logs/`. Support asks: "How many failed login attempts came from IP 10.2.3.4 in the last 30 days?"

## Diagram
```
File System approach:                 DBMS approach:
/var/logs/2026-07-01.csv               SELECT COUNT(*) FROM logs
/var/logs/2026-07-02.csv       vs      WHERE ip = '10.2.3.4'
...                                      AND ts > NOW() - INTERVAL '30 days';
/var/logs/2026-07-15.csv
  -> open 30 files, parse each,          -> index on (ip, ts), single query,
     grep line by line                       milliseconds
```

## Problem
Compare the file-system approach and DBMS approach for this query along these axes: query speed, concurrent access safety, data integrity constraints, and ease of adding a new query later (e.g. "top 10 IPs by failure count").

## Solution
- Query speed: file system requires opening/parsing every file (O(total lines)); a DBMS with an index on `(ip, ts)` can jump directly to matching rows.
- Concurrent access: multiple processes appending to the same CSV can interleave writes and corrupt rows; a DBMS serializes writes via transactions/locks.
- Integrity: nothing stops a bad log line with a malformed IP from being written to a CSV; a DBMS can enforce column types and constraints.
- New queries: "top 10 IPs by failure count" is one `GROUP BY` + `ORDER BY` + `LIMIT` in SQL versus writing a new parsing script in the file-system world.

## Takeaway
File systems are fine for opaque blobs (images, backups). Once you need structured queries, concurrent writers, and integrity guarantees, use a DBMS.
