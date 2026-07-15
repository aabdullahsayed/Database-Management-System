# DBMS Practice

A practical, problem-driven walk through database concepts, written the way a
software engineer actually runs into them - as bugs, incidents, design
reviews, and "why is this query slow" moments, not abstract definitions.

Every file follows the same shape:

```
# Title
## Scenario     - a concrete situation a SWE hits at work
## Diagram       - ASCII diagram where it helps
## Problem       - the explicit task/question
## Solution      - worked answer, with SQL/code where relevant
## Takeaway      - the one-line principle to remember
```

## How to use this

Read folders roughly in order (01 -> 16) if you're building up from scratch.
If you already know the basics, jump straight to whatever's biting you right
now - each file is self-contained.

## Structure

| Folder | Topic |
|---|---|
| 01-Introduction | What a DBMS is and why it beats flat files |
| 02-Relational-Model | Tables, tuples, keys |
| 03-ER-Model | Entities, relationships, cardinality, ER -> schema |
| 04-Normalization | Functional dependencies through BCNF, decomposition |
| 05-SQL-Basics | CREATE/INSERT/UPDATE/DELETE/SELECT fundamentals |
| 06-SQL-Intermediate | Aggregates, GROUP BY/HAVING, all JOIN types, subqueries |
| 07-SQL-Advanced | CTEs, recursive CTEs, window functions, views, procedures, triggers |
| 08-Indexing | B-Tree/B+Tree, clustered vs non-clustered, composite/covering indexes |
| 09-Transactions | ACID, commit/rollback/savepoint |
| 10-Concurrency-Control | Anomalies, locking, 2PL, deadlock handling |
| 11-Recovery-System | WAL, checkpoints, undo/redo, shadow paging |
| 12-Query-Processing | Parsing, optimization, execution plans, cost-based optimization |
| 13-Storage | File organization, heap/sequential/hash files, buffer management |
| 14-Distributed-DBMS | Fragmentation, replication, 2PC |
| 15-NoSQL | Key-value, document, column-family, graph, CAP theorem |
| 16-Case-Studies | Full schema designs: library, hospital, ecommerce, banking, social media |

110 files total, each a standalone practical problem with a worked solution.
