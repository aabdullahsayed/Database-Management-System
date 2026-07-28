# 10. Indexing & Query Optimization

## Quick Refresher
- An **index** is a separate data structure (usually a B-Tree) that lets the database find rows fast, without scanning the whole table — like a book's index.
- **Clustered index**: determines the physical storage order of table rows (a table can have only one).
- **Non-clustered index**: a separate structure pointing back to the actual row location (a table can have many).
- Indexes speed up **reads** (SELECT/WHERE/JOIN/ORDER BY) but slow down **writes** (INSERT/UPDATE/DELETE), since the index must be updated too.

## Practice Problems

### Q1 (Basic). Why would you add an index to a column frequently used in WHERE clauses?
**Answer:** Without an index, the database must scan every row (a "full table scan," O(n)) to check the condition. With an index (typically a B-Tree), it can jump almost directly to matching rows (roughly O(log n)), dramatically speeding up large tables.

### Q2 (Basic). What's the downside of adding too many indexes to a table?
**Answer:** Every `INSERT`, `UPDATE`, or `DELETE` must also update every index on that table, slowing down writes and increasing storage usage. Indexes are a trade-off: faster reads, slower writes, more disk space.

### Q3 (Intermediate). What is a composite (multi-column) index, and why does column ORDER matter?
**Answer:** A composite index on `(dept_id, salary)` is sorted first by `dept_id`, then by `salary` within each `dept_id`. It efficiently supports queries filtering on `dept_id` alone, or on `dept_id AND salary` together — but **not** efficiently on `salary` alone, because the index isn't globally sorted by salary. Rule of thumb: put the most selective / most commonly filtered-alone column **first**.

### Q4 (Intermediate). What is a covering index?
**Answer:** An index that contains **all** the columns needed by a query (in the SELECT, WHERE, and ORDER BY), so the database can answer the query directly from the index itself without going back to the actual table rows ("index-only scan") — significantly faster.

### Q5 (Intermediate). Why doesn't `WHERE UPPER(emp_name) = 'ALICE'` use a normal index on `emp_name`, even if one exists?
**Answer:** Applying a function (`UPPER()`) to the indexed column means the database can't directly compare the raw index values to the search term — the index is sorted by the original values, not their uppercased versions. Fix: either create a **functional/expression index** on `UPPER(emp_name)`, or store a normalized column (e.g., `emp_name_upper`) and index that instead.

### Q6 (Advanced/Interview). You run `EXPLAIN` on a slow query and see a "Full Table Scan" even though there's an index on the filtered column. List possible reasons why the optimizer might skip the index.
**Answer:**
- The query returns a **large fraction** of the table (e.g., >20-30%) — a full scan can actually be faster than jumping around via an index (avoiding many random-access disk reads).
- The column has **low selectivity** (e.g., a boolean `is_active` with mostly the same value) — the index doesn't narrow things down much.
- A function or type mismatch is applied to the column in the WHERE clause (see Q5), preventing index usage.
- The table's statistics are **stale**, causing the optimizer to misjudge how selective the filter really is.
- The query uses a **leading wildcard** in a LIKE (`LIKE '%smith'`) — B-Tree indexes can't efficiently search from the middle/end of a string.

### Q7 (Advanced/Interview). Explain the difference between a B-Tree index and a Hash index, and when you'd prefer one over the other.
**Answer:**
- **B-Tree**: keeps data sorted, supports range queries (`<`, `>`, `BETWEEN`), prefix matching (`LIKE 'abc%'`), and ordering (`ORDER BY`). Slightly slower than hash for pure equality lookups. This is the default and by far the most common index type.
- **Hash index**: extremely fast O(1) average equality lookups (`=`), but **cannot** support range queries or sorting at all, since hashed values have no meaningful order.
- **Choose Hash** only for pure equality-lookup workloads on a column never used in range queries or sorting (rare in practice); **choose B-Tree** as the default for almost everything else.

### Q8 (Advanced/Interview). A table has 10 million rows. A query filters `WHERE status = 'active' AND created_at > '2024-01-01' ORDER BY created_at DESC LIMIT 20`. Design an index strategy and explain your reasoning.
**Answer:** Create a composite index on `(status, created_at)`. Reasoning:
- `status` first, since it's an equality filter — the index can jump directly to the "active" segment.
- `created_at` second, since within that segment the rows are now sorted by `created_at`, which satisfies **both** the range filter (`> '2024-01-01'`) **and** the `ORDER BY created_at DESC` requirement — the database can read matching rows already in the correct sorted order and stop after just 20 rows, without a separate sort step.
- This avoids a costly full scan + explicit sort, turning the query into an efficient index range scan with an early exit at `LIMIT 20`.
