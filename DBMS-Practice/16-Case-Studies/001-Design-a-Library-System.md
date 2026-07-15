# Design a Library System

## Scenario
A university library needs a system to track books (some titles have multiple physical copies), members, and loans - including overdue tracking and a hold/reservation queue for popular books.

## Requirements
- A title (e.g. "Clean Code") can have multiple physical copies.
- A member can borrow multiple copies but only one copy of a given physical item at a time.
- Members can place holds on a title when all copies are checked out.
- Track due dates and overdue fines.

## Diagram
```
Book               BookCopy                  Member              Loan                 Hold
+---------+  1   M  +-----------+     M   M   +--------+   1   M  +------+    1   M    +------+
| book_id |------->| copy_id    |             | member |          | loan |             | hold |
| title   |         | book_id FK |             | _id    |         | _id  |             | _id  |
| author  |         | status     |             | name   |         +------+             +------+
+---------+         +-----------+             +--------+
```

## Schema
```sql
CREATE TABLE Book (
    book_id SERIAL PRIMARY KEY,
    title   VARCHAR(200) NOT NULL,
    author  VARCHAR(100) NOT NULL,
    isbn    VARCHAR(20) UNIQUE
);

CREATE TABLE BookCopy (
    copy_id SERIAL PRIMARY KEY,
    book_id INT NOT NULL REFERENCES Book(book_id),
    status  VARCHAR(20) NOT NULL DEFAULT 'available'  -- available, checked_out, lost
);

CREATE TABLE Member (
    member_id SERIAL PRIMARY KEY,
    name      VARCHAR(100) NOT NULL,
    email     VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE Loan (
    loan_id     SERIAL PRIMARY KEY,
    copy_id     INT NOT NULL REFERENCES BookCopy(copy_id),
    member_id   INT NOT NULL REFERENCES Member(member_id),
    borrowed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    due_at      TIMESTAMPTZ NOT NULL,
    returned_at TIMESTAMPTZ,       -- NULL while still checked out
    UNIQUE (copy_id, returned_at)  -- (with a partial index WHERE returned_at IS NULL)
);

CREATE TABLE Hold (
    hold_id    SERIAL PRIMARY KEY,
    book_id    INT NOT NULL REFERENCES Book(book_id),
    member_id  INT NOT NULL REFERENCES Member(member_id),
    placed_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (book_id, member_id)
);
```

## Key design decisions
- `Book` (the title/metadata) is separate from `BookCopy` (each physical, loanable item) - this models "one title, many copies" correctly, matching the actual real-world entities.
- A copy can only be actively loaned once at a time; enforced via a partial unique index on `(copy_id) WHERE returned_at IS NULL` (Postgres syntax) rather than trying to jam it into a plain unique constraint.
- Holds reference `book_id` (the title), not `copy_id` - a member wants "the next available copy of this title," not a specific physical copy.
- Overdue fines are computed from `due_at` vs `returned_at` (or `now()` if still out) rather than stored as a stale column, avoiding data that goes stale the moment a fine changes.

## Takeaway
The core modeling insight is separating the *conceptual* entity (a title) from its *physical instances* (copies) - a very common pattern (also applies to product SKUs vs. inventory units, event types vs. ticket instances).
