# DBMS Interview Practice Questions

> A curated collection of DBMS interview questions with detailed answers, organized by topic difficulty. Perfect for last-minute revision and deep-dive preparation.

---

## Table of Contents
1. [Basics & Fundamentals](#1-basics--fundamentals)
2. [SQL Queries](#2-sql-queries)
3. [Keys & Constraints](#3-keys--constraints)
4. [Normalization](#4-normalization)
5. [Transactions & ACID](#5-transactions--acid)
6. [Joins & Subqueries](#6-joins--subqueries)
7. [Indexing](#7-indexing)
8. [Concurrency Control](#8-concurrency-control)
9. [NoSQL vs SQL](#9-nosql-vs-sql)
10. [System Design & Advanced](#10-system-design--advanced)

---

## 1. Basics & Fundamentals

### Q1. What is a DBMS? How is it different from a File System?
**Answer:**
A Database Management System (DBMS) is software that enables users to create, manage, and manipulate databases. Unlike a file system, a DBMS provides:
- **Data Abstraction**: Hides physical storage details
- **Data Independence**: Logical and physical separation
- **Concurrency Control**: Multiple users can access data simultaneously
- **Data Integrity**: Enforces rules and constraints
- **Query Language**: SQL for structured data manipulation
- **Backup & Recovery**: Built-in mechanisms for data safety

| Feature | File System | DBMS |
|---------|------------|------|
| Redundancy | High | Low (normalized) |
| Data Sharing | Difficult | Easy |
| Integrity | Manual | Automatic |
| Security | OS-level | Granular |
| Querying | Application code | Declarative SQL |

---

### Q2. Explain the three levels of data abstraction in DBMS.
**Answer:**
1. **Physical Level**: Lowest level; describes HOW data is stored (indexes, file organization, compression).
2. **Logical Level**: Describes WHAT data is stored and relationships (tables, schemas, constraints).
3. **View Level**: Highest level; customized user views hiding complexity (virtual tables, access restrictions).

**Data Independence:**
- **Physical**: Change storage without affecting logical schema
- **Logical**: Change schema without affecting applications

---

### Q3. What are the different types of DBMS architectures?
**Answer:**
- **1-Tier**: Client, server, and database on same machine (e.g., local SQLite)
- **2-Tier**: Client directly connects to database (thick client)
- **3-Tier**: Client → Application Server → Database Server (web apps)
- **n-Tier**: Multiple middleware layers for scalability

---

### Q4. What is a Schema? Difference between Schema and Instance?
**Answer:**
- **Schema**: Blueprint/structure of the database (design-time). Analogous to a class in OOP.
- **Instance**: Actual data at a particular moment (run-time). Analogous to an object.

```
Schema:  Student(roll_no INT, name VARCHAR(50), age INT)
Instance: (1, 'Alice', 20), (2, 'Bob', 21)
```

---

## 2. SQL Queries

### Q5. What is the difference between `WHERE` and `HAVING`?
**Answer:**
| WHERE | HAVING |
|-------|--------|
| Filters rows BEFORE grouping | Filters groups AFTER aggregation |
| Cannot use aggregate functions | Can use aggregate functions |
| Applied to individual rows | Applied to grouped results |

```sql
-- Find departments with avg salary > 50000
SELECT dept, AVG(salary) 
FROM employees 
WHERE salary > 30000      -- filters rows first
GROUP BY dept 
HAVING AVG(salary) > 50000; -- filters groups
```

---

### Q6. Difference between `DELETE`, `TRUNCATE`, and `DROP`.
**Answer:**
| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| WHERE clause | Yes | No | No |
| Rollback | Possible | Not possible* | Not possible |
| Speed | Slow (row by row) | Fast (deallocate pages) | Fastest |
| Triggers | Fires | Does not fire | N/A |
| Table structure | Preserved | Preserved | Removed |
| Identity reset | No | Yes (usually) | N/A |

*In some DBs like PostgreSQL, TRUNCATE is transactional.

---

### Q7. What are Window Functions? Give examples.
**Answer:**
Window functions perform calculations across a set of rows related to the current row without collapsing them into a single output row.

```sql
-- Rank employees by salary within each department
SELECT 
    name, 
    dept, 
    salary,
    RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) as dense_rank,
    ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) as row_num,
    AVG(salary) OVER (PARTITION BY dept) as dept_avg,
    salary - AVG(salary) OVER (PARTITION BY dept) as diff_from_avg
FROM employees;
```

**Common Window Functions:**
- `ROW_NUMBER()`: Unique sequential number
- `RANK()`: Same rank for ties, skips next ranks
- `DENSE_RANK()`: Same rank for ties, no gaps
- `LEAD()/LAG()`: Access subsequent/previous rows
- `NTILE(n)`: Divide into n buckets

---

### Q8. Write a query to find the Nth highest salary.
**Answer:**
```sql
-- Method 1: Using LIMIT/OFFSET
SELECT DISTINCT salary 
FROM employees 
ORDER BY salary DESC 
LIMIT 1 OFFSET N-1;

-- Method 2: Using subquery with COUNT
SELECT DISTINCT e1.salary 
FROM employees e1 
WHERE N-1 = (SELECT COUNT(DISTINCT e2.salary) 
               FROM employees e2 
               WHERE e2.salary > e1.salary);

-- Method 3: Using DENSE_RANK (handles duplicates correctly)
SELECT salary 
FROM (
    SELECT DISTINCT salary, 
           DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
    FROM employees
) ranked 
WHERE rnk = N;
```

---

### Q9. Find duplicate records in a table.
**Answer:**
```sql
-- Find duplicates based on email
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Get all duplicate rows with IDs
SELECT u1.*
FROM users u1
JOIN users u2 ON u1.email = u2.email AND u1.id > u2.id;

-- Delete duplicates keeping one (using CTE)
WITH CTE AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn
    FROM users
)
DELETE FROM CTE WHERE rn > 1;
```

---

## 3. Keys & Constraints

### Q10. Explain all types of Keys in DBMS.
**Answer:**
1. **Super Key**: Any set of attributes that uniquely identifies a tuple (superset of candidate keys).
2. **Candidate Key**: Minimal super key (no redundant attributes). A table can have multiple.
3. **Primary Key**: Chosen candidate key. Unique + NOT NULL. Only one per table.
4. **Alternate Key**: Candidate keys not chosen as primary key.
5. **Foreign Key**: Establishes relationship between tables. References primary key of another table.
6. **Composite Key**: Key consisting of multiple attributes.
7. **Unique Key**: Ensures uniqueness but allows one NULL (usually).

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    email VARCHAR(100) UNIQUE,  -- Unique key
    FOREIGN KEY (customer_id) REFERENCES Customers(id)
);
```

---

### Q11. Can a table have multiple Primary Keys? Multiple Foreign Keys?
**Answer:**
- **Primary Keys**: NO. Only ONE primary key per table (can be composite).
- **Foreign Keys**: YES. A table can have multiple foreign keys referencing different tables.

```sql
CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),  -- Composite PK
    FOREIGN KEY (student_id) REFERENCES Students(id),
    FOREIGN KEY (course_id) REFERENCES Courses(id)
);
```

---

### Q12. What is the difference between `UNIQUE` and `PRIMARY KEY`?
**Answer:**
| Feature | PRIMARY KEY | UNIQUE |
|---------|-------------|--------|
| NULL allowed | No | Yes (one NULL usually) |
| Count per table | One | Multiple |
| Purpose | Entity identification | Business rule enforcement |
| Clustered index | Yes (usually) | No (usually non-clustered) |

---

## 4. Normalization

### Q13. What is Normalization? Explain 1NF, 2NF, 3NF, BCNF.
**Answer:**
Normalization is the process of organizing data to minimize redundancy and dependency.

**1NF (First Normal Form):**
- Atomic values (no multi-valued attributes)
- No repeating groups
- Each cell contains single value

```
Before: Student(id, name, subjects)  -- subjects = "Math, Physics"
After:  Student_Subject(student_id, subject)
```

**2NF (Second Normal Form):**
- Must be in 1NF
- No partial dependency (non-prime attributes depend on FULL candidate key)
- Relevant for composite keys

```
Before: Enrollment(student_id, course_id, student_name, course_name, grade)
After:  Student(student_id, student_name)
        Course(course_id, course_name)
        Enrollment(student_id, course_id, grade)
```

**3NF (Third Normal Form):**
- Must be in 2NF
- No transitive dependency (non-prime attributes depend only on candidate key)

```
Before: Employee(emp_id, emp_name, dept_id, dept_name, dept_location)
After:  Employee(emp_id, emp_name, dept_id)
        Department(dept_id, dept_name, dept_location)
```

**BCNF (Boyce-Codd Normal Form):**
- Stricter than 3NF
- For every FD X → Y, X must be a superkey
- Handles anomalies 3NF misses

```
Before: Student(subject, professor, student) 
        -- professor determines subject, but professor is not superkey
After:  Professor_Subject(professor, subject)
        Enrollment(student, professor)
```

---

### Q14. What are the anomalies that normalization prevents?
**Answer:**
1. **Insertion Anomaly**: Cannot insert data without unrelated data
   - *Example*: Can't add a new department without having an employee
2. **Update Anomaly**: Updating one record requires updating multiple rows
   - *Example*: Changing department name in every employee record
3. **Deletion Anomaly**: Deleting a record loses unrelated information
   - *Example*: Deleting last employee deletes department info

---

### Q15. When would you DENORMALIZE a database?
**Answer:**
Denormalization intentionally adds redundancy for performance gains:
- **Read-heavy workloads**: Pre-computed joins reduce query time
- **Reporting/OLAP**: Star schemas for analytics
- **Caching**: Storing computed values (e.g., total_order_amount)
- **Microservices**: Each service owns its data copy
- **Trade-off**: Faster reads vs. slower writes and data inconsistency risk

---

## 5. Transactions & ACID

### Q16. Explain ACID properties with examples.
**Answer:**
**A - Atomicity**: All operations complete or none. "All or nothing."
```
Bank transfer: Debit + Credit must both succeed or both fail.
```

**C - Consistency**: Database moves from one valid state to another.
```
Total money before = Total money after transfer.
```

**I - Isolation**: Concurrent transactions don't interfere.
```
Transaction A reading balance shouldn't see uncommitted changes from Transaction B.
```

**D - Durability**: Committed changes survive system failures.
```
After COMMIT, data is written to disk/WAL even if power fails.
```

---

### Q17. What are Transaction States?
**Answer:**
```
        [Active]
          |
    +-----+-----+
    |           |
[Partially    [Failed]
 Committed]      |
    |         [Aborted]
    |            |
[Committed]  [Terminated]
```
- **Active**: Transaction is executing
- **Partially Committed**: Final statement executed, waiting for commit
- **Committed**: Successfully completed
- **Failed**: Normal execution can't proceed
- **Aborted**: Rolled back, database restored to prior state
- **Terminated**: End of transaction lifecycle

---

### Q18. Explain Isolation Levels and their problems.
**Answer:**
| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-----------------|------------|---------------------|--------------|
| READ UNCOMMITTED | ✓ Allowed | ✓ Allowed | ✓ Allowed |
| READ COMMITTED | ✗ Prevented | ✓ Allowed | ✓ Allowed |
| REPEATABLE READ | ✗ Prevented | ✗ Prevented | ✓ Allowed* |
| SERIALIZABLE | ✗ Prevented | ✗ Prevented | ✗ Prevented |

\* In MySQL InnoDB, REPEATABLE READ also prevents phantom reads using MVCC + gap locks.

**Problems:**
- **Dirty Read**: Reading uncommitted data
- **Non-Repeatable Read**: Same query returns different data within transaction
- **Phantom Read**: New rows appear/disappear between queries

---

### Q19. What is a Deadlock? How to prevent it?
**Answer:**
A deadlock occurs when two or more transactions wait indefinitely for each other to release locks.

```
T1: Lock A → Request Lock B
T2: Lock B → Request Lock A
     [DEADLOCK]
```

**Prevention Strategies:**
1. **Lock Ordering**: Always acquire locks in a predefined order
2. **Timeout**: Abort transaction if lock not acquired within time
3. **Wait-Die** (Non-preemptive): Older transaction waits, younger dies
4. **Wound-Wait** (Preemptive): Older wounds (aborts) younger, younger waits
5. **Banker's Algorithm**: Resource allocation graph analysis

**Detection:** Wait-for graph cycle detection.

---

## 6. Joins & Subqueries

### Q20. Explain all types of JOINs with Venn diagram logic.
**Answer:**
```sql
-- INNER JOIN: Only matching rows from both tables
SELECT * FROM A INNER JOIN B ON A.id = B.id;
-- Venn: Intersection of A and B

-- LEFT JOIN: All rows from A, matching from B (NULL if no match)
SELECT * FROM A LEFT JOIN B ON A.id = B.id;
-- Venn: Entire A circle

-- RIGHT JOIN: All rows from B, matching from A
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;
-- Venn: Entire B circle

-- FULL OUTER JOIN: All rows from both (NULL where no match)
SELECT * FROM A FULL OUTER JOIN B ON A.id = B.id;
-- Venn: Union of A and B

-- CROSS JOIN: Cartesian product (every row of A with every row of B)
SELECT * FROM A CROSS JOIN B;
-- Venn: Not applicable - all combinations

-- SELF JOIN: Table joined with itself
SELECT e.name, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

### Q21. Difference between Correlated and Non-Correlated Subqueries.
**Answer:**
**Non-Correlated**: Independent, executes once.
```sql
SELECT * FROM employees 
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Correlated**: Depends on outer query, executes for each row.
```sql
SELECT e1.name, e1.salary
FROM employees e1
WHERE salary > (SELECT AVG(salary) 
                FROM employees e2 
                WHERE e2.dept = e1.dept);  -- references outer query
```

| Feature | Non-Correlated | Correlated |
|---------|---------------|------------|
| Execution | Once | Per outer row |
| Performance | Better | Slower |
| Dependency | Independent | References outer table |

---

### Q22. What is an EXISTS clause? When to use it over IN?
**Answer:**
```sql
-- EXISTS: Returns true if subquery returns any row (stops at first match)
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- IN: Checks membership in a set
SELECT * FROM departments
WHERE id IN (SELECT DISTINCT dept_id FROM employees);
```

**Use EXISTS when:**
- Subquery is large (stops at first match)
- Checking for NULLs (IN with NULL gives unexpected results)
- Correlated subquery scenario

**Use IN when:**
- Subquery is small and indexed
- Comparing against a fixed list of values

---

## 7. Indexing

### Q23. What is an Index? Why and when to use it?
**Answer:**
An index is a data structure that improves query speed at the cost of storage and write performance.

**Types:**
- **B-Tree Index**: Default, balanced tree, good for range queries
- **Hash Index**: Exact match only, O(1) lookup
- **Bitmap Index**: Low cardinality columns (gender, status)
- **Full-Text Index**: Text search
- **Spatial Index**: Geographic data
- **Composite Index**: Multiple columns

**When to use:**
- Columns in WHERE, JOIN, ORDER BY
- High cardinality columns
- Read-heavy tables

**When NOT to use:**
- Small tables
- Frequently updated columns
- Low cardinality columns (unless bitmap)

---

### Q24. What is the difference between Clustered and Non-Clustered Index?
**Answer:**
| Feature | Clustered Index | Non-Clustered Index |
|---------|----------------|---------------------|
| Data order | Determines physical order | Logical order only |
| Count per table | Only ONE | Multiple (usually 249+) |
| Leaf nodes | Contain actual data | Contain pointers to data |
| Speed | Faster for range queries | Slower (extra lookup) |
| Storage | No extra space | Extra space required |
| Default | Primary key | UNIQUE constraints |

```
Clustered:    [10] → [20] → [30]  (data stored in leaf)
Non-Clustered: [10] → ptr → [20] → ptr → [30] → ptr
```

---

### Q25. What is a Covering Index?
**Answer:**
An index that contains ALL columns needed for a query, so the database doesn't need to access the actual table.

```sql
-- Query
SELECT name, salary FROM employees WHERE dept = 'Engineering';

-- Covering Index
CREATE INDEX idx_covering ON employees(dept, name, salary);
-- All needed columns are in the index → Index Only Scan
```

**Benefits:**
- Eliminates table lookups
- Reduces I/O
- Faster query execution

---

### Q26. Explain Index Cardinality and Selectivity.
**Answer:**
- **Cardinality**: Number of unique values in a column
- **Selectivity**: Cardinality / Total Rows (ratio of unique values)

```
High Selectivity (> 0.1): Good for indexing (email, SSN)
Low Selectivity (< 0.01): Poor for B-Tree (gender, boolean)
```

**Formula:**
```
Selectivity = COUNT(DISTINCT column) / COUNT(*)
```

---

## 8. Concurrency Control

### Q27. What is MVCC (Multi-Version Concurrency Control)?
**Answer:**
MVCC maintains multiple versions of data to handle concurrent access without excessive locking.

**How it works:**
1. Each transaction gets a timestamp/transaction ID
2. Writes create new versions instead of overwriting
3. Readers see a consistent snapshot from transaction start time
4. Old versions are cleaned up by vacuum processes

**Benefits:**
- Readers don't block writers
- Writers don't block readers
- Better concurrency than strict locking

**Used by:** PostgreSQL, MySQL InnoDB, Oracle

---

### Q28. Explain Lock Granularity and Lock Escalation.
**Answer:**
**Lock Granularity Levels:**
- **Row-level**: Finest, highest concurrency, most overhead
- **Page-level**: Intermediate (8KB blocks)
- **Table-level**: Coarsest, lowest concurrency, least overhead

**Lock Escalation:**
When a transaction acquires too many row locks, the DBMS may escalate to a table lock to reduce overhead.

```sql
-- SQL Server example
-- After ~5000 row locks on a table → escalated to table lock
```

**Trade-offs:**
- Fine granularity = Better concurrency, More memory
- Coarse granularity = Worse concurrency, Less overhead

---

### Q29. What are Shared Locks and Exclusive Locks?
**Answer:**
| Lock Type | Symbol | Allows Read | Allows Write | Compatible With |
|-----------|--------|-------------|--------------|-----------------|
| Shared (S) | S | Yes | No | S |
| Exclusive (X) | X | No | No | None |
| Update (U) | U | Yes | No (until converted) | S |
| Intent Shared (IS) | IS | - | - | IS, IX, S |
| Intent Exclusive (IX) | IX | - | - | IX, IS |

**Compatibility Matrix:**
```
        S   X   IS  IX
S       ✓   ✗   ✓   ✗
X       ✗   ✗   ✗   ✗
IS      ✓   ✗   ✓   ✓
IX      ✗   ✗   ✓   ✓
```

---

## 9. NoSQL vs SQL

### Q30. Compare SQL and NoSQL databases.
**Answer:**
| Feature | SQL (Relational) | NoSQL (Non-Relational) |
|---------|------------------|------------------------|
| Schema | Fixed, predefined | Flexible, dynamic |
| Scaling | Vertical (scale-up) | Horizontal (scale-out) |
| Data Model | Tables (rows/columns) | Document, Key-Value, Graph, Column |
| ACID | Full ACID support | BASE (Basically Available, Soft state, Eventual consistency) |
| Joins | Native support | Application-side or limited |
| Use Case | Complex queries, transactions | Big data, real-time, unstructured |
| Examples | MySQL, PostgreSQL, Oracle | MongoDB, Cassandra, Redis, Neo4j |

---

### Q31. When to choose NoSQL over SQL?
**Answer:**
Choose **NoSQL** when:
- Massive scale (petabytes of data)
- Need for horizontal scaling across commodity hardware
- Unstructured/semi-structured data (JSON, logs)
- High write throughput (time-series, IoT)
- Rapid prototyping with evolving schema
- Geo-distributed deployments

Choose **SQL** when:
- Complex joins and aggregations
- Strong consistency requirements (banking, inventory)
- ACID transactions are critical
- Well-defined schema with relationships
- Need for ad-hoc analytical queries

---

### Q32. Explain CAP Theorem.
**Answer:**
In a distributed data store, you can only guarantee TWO of the three:

- **C - Consistency**: All nodes see the same data at the same time
- **A - Availability**: Every request receives a response (success or failure)
- **P - Partition Tolerance**: System continues despite network partitions

**Combinations:**
- **CP** (Consistency + Partition Tolerance): MongoDB, HBase, Redis Cluster
- **AP** (Availability + Partition Tolerance): Cassandra, DynamoDB, CouchDB
- **CA** (Consistency + Availability): Traditional single-node RDBMS

> **Important**: In distributed systems, partition tolerance is mandatory. So the real choice is between CP and AP.

---

## 10. System Design & Advanced

### Q33. What is Sharding? Different sharding strategies.
**Answer:**
Sharding splits a large database into smaller, manageable pieces (shards) distributed across servers.

**Strategies:**
1. **Hash Sharding**: `shard = hash(key) % N`
   - Even distribution
   - Rebalancing needed when N changes

2. **Range Sharding**: Data divided by key ranges
   - Good for range queries
   - Hotspot risk (recent data)

3. **Directory-Based**: Lookup service maps keys to shards
   - Flexible, but lookup service is a bottleneck

4. **Geo-Sharding**: Data partitioned by geography
   - Low latency for local users

**Challenges:**
- Cross-shard joins are expensive
- Rebalancing when adding shards
- Shard key selection is critical

---

### Q34. What is Database Replication? Types?
**Answer:**
Replication copies data across multiple servers for availability and scalability.

**Types by Role:**
- **Master-Slave (Primary-Replica)**: Writes to master, reads from slaves
- **Master-Master**: Both accept writes (conflict resolution needed)
- **Multi-Master**: Multiple primaries (complex)

**Types by Sync:**
- **Synchronous**: Wait for all replicas to confirm (strong consistency, higher latency)
- **Asynchronous**: Fire-and-forget (better performance, risk of data loss)
- **Semi-Synchronous**: Wait for at least one replica

**Topologies:**
- Single leader
- Circular
- Star
- Mesh

---

### Q35. Explain the difference between OLTP and OLAP.
**Answer:**
| Feature | OLTP | OLAP |
|---------|------|------|
| Purpose | Transaction processing | Analytical processing |
| Queries | Simple, short, frequent | Complex, long, ad-hoc |
| Data Volume | GBs | TBs/PBs |
| Normalization | Highly normalized (3NF) | Denormalized (star/snowflake) |
| Users | Thousands (concurrent) | Few (analysts) |
| Writes | Heavy, real-time | Batch loads |
| Examples | Order entry, banking | Data warehousing, BI |
| Schema | Entity-Relationship | Star/Snowflake |

---

### Q36. What is a Materialized View?
**Answer:**
A materialized view stores the RESULT of a query physically on disk, unlike a regular view which is just a stored query.

```sql
CREATE MATERIALIZED VIEW sales_summary AS
SELECT region, SUM(amount), COUNT(*) 
FROM sales 
GROUP BY region;

-- Refresh strategies
REFRESH MATERIALIZED VIEW sales_summary;  -- Manual
REFRESH MATERIALIZED VIEW CONCURRENTLY sales_summary;  -- Without locks
```

**Benefits:**
- Faster query performance for expensive aggregations
- Pre-computed results

**Trade-offs:**
- Stale data until refreshed
- Storage overhead
- Refresh cost

---

### Q37. How would you design a URL shortener database schema?
**Answer:**
```sql
-- URLs table
CREATE TABLE urls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(10) UNIQUE NOT NULL,  -- e.g., 'aB3xK9'
    long_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NULL,
    user_id INT,
    click_count BIGINT DEFAULT 0,
    INDEX idx_short_code (short_code)
);

-- Analytics table (sharded by date)
CREATE TABLE url_clicks (
    id BIGINT PRIMARY KEY,
    url_id BIGINT NOT NULL,
    clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    country_code CHAR(2),
    referrer TEXT,
    FOREIGN KEY (url_id) REFERENCES urls(id)
);

-- Users table (optional)
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    api_key VARCHAR(64) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Design Considerations:**
- Use base62 encoding for short codes (a-z, A-Z, 0-9)
- Partition `url_clicks` by date for scalability
- Cache hot URLs in Redis
- Use consistent hashing for sharding

---

### Q38. How to handle database migrations in production?
**Answer:**
**Best Practices:**
1. **Backward Compatibility**: New code must work with old schema
2. **Expand-Contract Pattern**:
   - Phase 1: Add new column/table (expand)
   - Phase 2: Update application to write to both
   - Phase 3: Backfill data
   - Phase 4: Switch reads to new schema
   - Phase 5: Remove old column (contract)

3. **Online Schema Changes**: Use tools like:
   - **pt-online-schema-change** (Percona)
   - **gh-ost** (GitHub)
   - **pg_repack** (PostgreSQL)

4. **Blue-Green Deployment**: Migrate on green, switch traffic

5. **Rollback Plan**: Always have a way back

---

### Q39. What are database connection pooling and why is it important?
**Answer:**
Connection pooling maintains a cache of database connections that can be reused.

**Why important:**
- Creating connections is expensive (TCP handshake + auth)
- Prevents connection exhaustion
- Reduces latency for application requests
- Better resource management

**Popular Pools:**
- Java: HikariCP, C3P0
- Python: SQLAlchemy pool, psycopg2 pool
- Node.js: pg-pool, generic-pool

**Key Parameters:**
- `min_connections`: Baseline pool size
- `max_connections`: Upper limit
- `connection_timeout`: Max wait for available connection
- `idle_timeout`: Close unused connections

---

### Q40. Explain Write-Ahead Logging (WAL).
**Answer:**
WAL is a technique where changes are written to a log BEFORE they are applied to the database.

**Process:**
1. Transaction begins
2. Changes written to WAL (sequential, fast)
3. WAL flushed to disk
4. Changes applied to data files (random I/O, can be delayed)
5. Transaction commits when WAL is durable

**Benefits:**
- **Durability**: Log ensures committed transactions survive crashes
- **Performance**: Sequential log writes vs random data file writes
- **Recovery**: Replay log to restore consistency after crash
- **Replication**: WAL shipping for standby servers

**Used by:** PostgreSQL, MySQL InnoDB (redo log), SQLite

---

## Bonus: Quick Revision Checklist

### 🔑 Key Concepts to Remember
- [ ] ACID properties and their implementation
- [ ] All normal forms (1NF → BCNF)
- [ ] Types of joins and when to use each
- [ ] Index types and selection criteria
- [ ] Isolation levels and anomalies
- [ ] Lock types and deadlock handling
- [ ] CAP theorem trade-offs
- [ ] Sharding vs Replication
- [ ] OLTP vs OLAP

### 📝 SQL Patterns to Practice
- [ ] Nth highest salary (3 methods)
- [ ] Running totals and moving averages (window functions)
- [ ] Pivot/unpivot data
- [ ] Tree/hierarchical queries (CTE recursion)
- [ ] Finding gaps and islands in sequences
- [ ] Top-N per group
- [ ] Median calculation

### 🏗️ Design Questions
- [ ] Design a Twitter-like feed
- [ ] Design an e-commerce cart system
- [ ] Design a rate limiter
- [ ] Design a leaderboard
- [ ] Design a messaging system

---

*Good luck with your interview! Remember to think out loud and discuss trade-offs.*
