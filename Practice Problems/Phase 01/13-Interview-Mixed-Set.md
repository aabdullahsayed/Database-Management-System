# 13. Interview-Grade Mixed Problem Set

A mixed bag of the kind of questions that show up in real DBMS interview rounds — combining SQL writing, schema reasoning, and "explain the trade-off" conceptual questions. Try each before reading the answer.

---

### Q1. Find the second-highest salary in the Employees table WITHOUT using LIMIT/OFFSET or TOP.
```sql
SELECT MAX(salary) AS second_highest
FROM Employees
WHERE salary < (SELECT MAX(salary) FROM Employees);
```
**Follow-up the interviewer might ask:** "What if there are ties for the highest salary?" — this query still works correctly because it filters by *value*, not row position.

---

### Q2. Find duplicate rows in a table (e.g., duplicate employee entries by name and department).
```sql
SELECT emp_name, dept_id, COUNT(*)
FROM Employees
GROUP BY emp_name, dept_id
HAVING COUNT(*) > 1;
```

---

### Q3. Delete duplicate rows, keeping only one copy of each (assume a surrogate `emp_id` exists, keep the lowest ID).
```sql
DELETE FROM Employees
WHERE emp_id NOT IN (
    SELECT MIN(emp_id)
    FROM Employees
    GROUP BY emp_name, dept_id, salary  -- columns defining a "duplicate"
);
```

---

### Q4. Find employees who earn more than their manager.
```sql
SELECT e.emp_name AS employee, m.emp_name AS manager
FROM Employees e
JOIN Employees m ON e.manager_id = m.emp_id
WHERE e.salary > m.salary;
```

---

### Q5. Find the department with the highest total salary spend.
```sql
SELECT dept_id, SUM(salary) AS total_spend
FROM Employees
GROUP BY dept_id
ORDER BY total_spend DESC
LIMIT 1;
```

---

### Q6. Pivot data: show total hours worked per employee per project as a cross-tab (employees as rows, project names as columns). Explain the general SQL approach (exact syntax varies by database).
**Answer (conceptual, using conditional aggregation — works across most SQL dialects):**
```sql
SELECT e.emp_name,
    SUM(CASE WHEN p.project_name = 'Alpha' THEN a.hours ELSE 0 END) AS Alpha,
    SUM(CASE WHEN p.project_name = 'Beta'  THEN a.hours ELSE 0 END) AS Beta
FROM Employees e
JOIN Assignments a ON e.emp_id = a.emp_id
JOIN Projects p ON a.project_id = p.project_id
GROUP BY e.emp_name;
```
**Key idea:** "pivoting" without a dedicated `PIVOT` keyword (which not all databases support) is done via `SUM(CASE WHEN ... THEN value ELSE 0 END)` per desired column — this is a very common interview trick question.

---

### Q7. Explain what happens, step by step, when you run `SELECT * FROM Employees WHERE dept_id = 3;` on a database with an index on `dept_id` (describe the query execution path).
**Answer:**
1. **Parser** checks SQL syntax and builds a parse tree.
2. **Query optimizer** considers possible execution plans (full scan vs. index scan) and picks the cheapest one based on table statistics (e.g., how selective `dept_id = 3` is).
3. If using the index: the engine performs an **index seek** on the B-Tree for `dept_id = 3`, retrieving matching row locations.
4. For each match, if it's a non-covering index, the engine does a **lookup back to the table** (heap or clustered index) to fetch the full row.
5. Rows are returned to the client, often streamed rather than all buffered at once.

---

### Q8. A junior developer wrote `SELECT * FROM Orders WHERE YEAR(order_date) = 2024;` and it's slow on a 50-million-row table with an index on `order_date`. Why, and how would you fix it?
**Answer:** Wrapping the indexed column in a function (`YEAR(...)`) prevents the database from using the index directly — same issue as Chapter 10 Q5. Fix: rewrite as a **range condition** on the raw column so the index can be used:
```sql
SELECT * FROM Orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

---

### Q9. Design question: You're asked to design a database schema for a simplified "Twitter" — users can post tweets, follow other users, and like tweets. Sketch the core tables and discuss one scaling challenge.
**Answer (sample):**
- `Users(user_id PK, username, ...)`
- `Tweets(tweet_id PK, user_id FK, content, created_at)`
- `Follows(follower_id FK, followee_id FK, PK(follower_id, followee_id))`
- `Likes(user_id FK, tweet_id FK, PK(user_id, tweet_id))`

**Scaling challenge:** Generating a user's home timeline ("tweets from everyone I follow, in order") is expensive to compute live via a join + sort for users following thousands of accounts. Real systems typically use a **fan-out-on-write** strategy — when a user tweets, push the tweet into each follower's precomputed timeline (often in a fast key-value/cache store) — trading extra write work for very fast reads, with special handling for "celebrity" accounts with millions of followers (where fan-out-on-write becomes too expensive, and a hybrid pull-based approach is used instead).

---

### Q10. Explain, in one paragraph, how you'd justify to a team choosing PostgreSQL vs. MongoDB for a new project's primary database — what questions would you ask first?
**Answer:** The key questions: How structured/predictable is the data model, and does it need multi-record transactions with strong consistency (favor PostgreSQL)? Or does it have many varying/nested attributes per record, high write throughput needs, and looser consistency requirements (favor MongoDB)? How important is complex relational querying/reporting/joins (favor PostgreSQL, which has decades of query optimizer maturity)? What's the team's existing expertise, and what's the expected data scale/growth pattern? There's no universally "better" choice — the right answer depends on the shape of the data and the read/write/consistency patterns of the actual application.

---

## Quick Self-Check List Before an Interview
- [ ] Can you explain ACID and CAP without mixing up the "C"s?
- [ ] Can you write a query using window functions to get "top N per group"?
- [ ] Can you explain 1NF/2NF/3NF/BCNF with your own example?
- [ ] Can you explain when an index would/wouldn't be used, and why?
- [ ] Can you explain the difference between optimistic and pessimistic locking?
- [ ] Can you design a simple schema from a vague prompt and explain your relationship choices out loud?
