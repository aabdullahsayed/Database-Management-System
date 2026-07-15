# Convert ER to Relational

## Scenario
Design review hands you this ER sketch for a blogging platform and asks you to produce the actual `CREATE TABLE` statements.

## Diagram
```
+--------+        1      M      +---------+       M      M      +--------+
| Author |---------------------<| Post    |>-------------------<|  Tag   |
+--------+       "writes"       +---------+     "tagged_with"    +--------+
| author_id PK|                 | post_id PK|                    | tag_id PK|
| name         |                 | title      |                    | name     |
+--------+                       | author_id FK|                  +--------+
                                 +---------+
```

## Problem
Convert this ER diagram into relational tables, handling the 1:M `Author`-`Post` relationship and the M:N `Post`-`Tag` relationship.

## Solution
```sql
CREATE TABLE Author (
    author_id INT PRIMARY KEY,
    name      VARCHAR(100) NOT NULL
);

CREATE TABLE Post (
    post_id   INT PRIMARY KEY,
    title     VARCHAR(200) NOT NULL,
    author_id INT NOT NULL,
    FOREIGN KEY (author_id) REFERENCES Author(author_id)
);

CREATE TABLE Tag (
    tag_id INT PRIMARY KEY,
    name   VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE PostTag (               -- junction table for M:N
    post_id INT NOT NULL,
    tag_id  INT NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES Post(post_id),
    FOREIGN KEY (tag_id)  REFERENCES Tag(tag_id)
);
```

## Takeaway
General rule for ER-to-relational: strong entities -> tables with their own PK; 1:M relationships -> FK on the "many" side; M:N relationships -> a new junction table with a composite PK of both FKs.
