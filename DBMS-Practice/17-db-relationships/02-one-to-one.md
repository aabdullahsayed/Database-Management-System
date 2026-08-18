# 2. One-to-One (1:1) Relationships

## The analogy: a person and their passport

Each person has **exactly one** passport. Each passport belongs to **exactly one** person. Neither side is ever shared. If you tried to give one passport to two people, immigration would have a very bad day.

That's a one-to-one relationship: **one row on side A matches at most one row on side B, and vice versa.**

## Why split it into two tables at all?

If it's strictly 1:1, why not just put everything in one table? Two common reasons:

1. **The extra data is optional or large**, and most queries don't need it. Example: `users` (queried constantly, on every request) vs. `user_profiles` (bio, avatar, address — only needed on the profile page). Keeping them separate keeps the frequently-used table lean and fast.
2. **Different access/security rules.** Example: `users` (public-ish) vs. `user_secrets` (password hash, 2FA seed — locked down, rarely joined).

## Table design

```sql
CREATE TABLE users (
    id    SERIAL PRIMARY KEY,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE user_profiles (
    id      SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE REFERENCES users(id),  -- UNIQUE is what makes this 1:1, not 1:N
    bio     TEXT,
    avatar_url TEXT
);
```

The critical detail: **`user_id` has a `UNIQUE` constraint.** Without `UNIQUE`, nothing stops ten profile rows from pointing at the same user — that would silently turn this into a one-to-many. The `UNIQUE` constraint is the database physically enforcing "one passport per person."

## Querying it

```sql
SELECT users.email, user_profiles.bio
FROM users
JOIN user_profiles ON user_profiles.user_id = users.id
WHERE users.id = 4821;
```

## Where the foreign key lives — does it matter?

You could instead put `user_id` on `users` pointing to `profiles.id`. Either direction works technically. Convention: put the FK on whichever table represents the **optional or dependent** side — the profile can't exist without a user, but a user can exist (briefly) without a profile. So the FK sits on `user_profiles`.

## When people get this wrong

A very common mistake: modeling something as 1:1 that's actually 1:N later. Example: "each user has one address" — until the product grows and users need a home address *and* a shipping address. At that point the `UNIQUE` constraint has to go, and it becomes a one-to-many (file 3). This is normal — model for what's true *today*, but know which constraint you'd remove if the shape changes.

Next: [`03-one-to-many.md`](03-one-to-many.md)
