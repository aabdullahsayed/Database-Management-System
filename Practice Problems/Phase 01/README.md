# DBMS Practice Problems — Basic to Advanced (Interview Grade) 🗄️

A structured set of DBMS practice problems, ordered from fundamentals to advanced interview-level topics. Each file has a short concept refresher, then practice problems with worked solutions.

## Contents
1. [ER Model & Database Design Basics](01-ER-Model-and-Design.md)
2. [Relational Model, Keys & Constraints](02-Relational-Model-and-Keys.md)
3. [SQL Basics — SELECT, WHERE, ORDER BY](03-SQL-Basics.md)
4. [SQL Joins](04-SQL-Joins.md)
5. [Aggregation, GROUP BY, HAVING](05-Aggregation-and-Grouping.md)
6. [Subqueries & Set Operations](06-Subqueries-and-Set-Operations.md)
7. [Normalization (1NF → BCNF → 4NF)](07-Normalization.md)
8. [Transactions & ACID Properties](08-Transactions-and-ACID.md)
9. [Concurrency Control & Locking](09-Concurrency-Control.md)
10. [Indexing & Query Optimization](10-Indexing-and-Optimization.md)
11. [Advanced SQL — Window Functions & CTEs](11-Advanced-SQL.md)
12. [NoSQL, CAP Theorem & Distributed DBs](12-NoSQL-and-CAP.md)
13. [Interview-Grade Mixed Problem Set](13-Interview-Mixed-Set.md)

## How to Use
- Work through the files in order — later chapters assume earlier concepts.
- Each problem has a **Question**, then a collapsible-style **Answer** right below it — cover the answer with your hand/scroll slowly and try it yourself first!
- A shared sample schema (used in the SQL files) is defined below so all queries are consistent.

## Sample Schema (used throughout SQL problems)

```sql
-- Employees working in departments
CREATE TABLE Departments (
    dept_id     INT PRIMARY KEY,
    dept_name   VARCHAR(50)
);

CREATE TABLE Employees (
    emp_id      INT PRIMARY KEY,
    emp_name    VARCHAR(50),
    dept_id     INT,
    manager_id  INT,
    salary      DECIMAL(10,2),
    hire_date   DATE,
    FOREIGN KEY (dept_id) REFERENCES Departments(dept_id),
    FOREIGN KEY (manager_id) REFERENCES Employees(emp_id)
);

CREATE TABLE Projects (
    project_id   INT PRIMARY KEY,
    project_name VARCHAR(50),
    dept_id      INT,
    budget       DECIMAL(12,2),
    FOREIGN KEY (dept_id) REFERENCES Departments(dept_id)
);

CREATE TABLE Assignments (
    emp_id     INT,
    project_id INT,
    hours      INT,
    PRIMARY KEY (emp_id, project_id),
    FOREIGN KEY (emp_id) REFERENCES Employees(emp_id),
    FOREIGN KEY (project_id) REFERENCES Projects(project_id)
);
```

Good luck! 🚀
