# Create Table

## Problem
Design and create a `users` table for a signup flow: unique email, hashed password, creation timestamp, and an active/inactive flag.

## Solution
```sql
CREATE TABLE users (
    id            BIGSERIAL PRIMARY KEY,
    email         VARCHAR(255) NOT NULL UNIQUE,
    password_hash CHAR(60)     NOT NULL,        -- e.g. bcrypt hash length
    is_active     BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT now()
);
```

Design notes:
- `BIGSERIAL` for the PK avoids running out of IDs at scale (regular `SERIAL`/int caps around 2.1 billion).
- `email UNIQUE NOT NULL` pushes the "one account per email" rule into the schema, not just application code.
- Never store plaintext passwords - only the hash, and never trust application code alone to enforce that; code review + schema comments help but this is a process discipline, not a schema-enforced one.
- `TIMESTAMPTZ` instead of `TIMESTAMP` avoids timezone bugs when your service scales across regions.

## Takeaway
A table definition is your first line of defense for data integrity - encode as many business rules (`NOT NULL`, `UNIQUE`, sensible types) into the schema as you can, rather than relying solely on application validation.
