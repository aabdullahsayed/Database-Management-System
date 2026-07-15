# Limit

## Scenario
You're building infinite-scroll pagination for a feed: "give me items 21-40" (page 3, 20 items per page).

## Problem
Write a paginated query for page 3 with a page size of 20, and note the performance pitfall of this approach at very large offsets.

## Solution
```sql
SELECT id, title, created_at
FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;   -- skip the first 40 rows (pages 1 and 2), take the next 20
```

**Pitfall:** `OFFSET` at large values (e.g. `OFFSET 1000000`) forces the database to scan and discard a million rows before it can return your 20 - it gets slower the deeper you paginate. For "infinite scroll" style pagination at scale, prefer **keyset/cursor pagination**:
```sql
SELECT id, title, created_at
FROM posts
WHERE created_at < '2026-07-10 12:00:00'   -- the created_at of the last item you saw
ORDER BY created_at DESC
LIMIT 20;
```

## Takeaway
`LIMIT/OFFSET` is fine for small, page-numbered UIs; for feeds/infinite scroll or large datasets, use keyset (cursor-based) pagination to keep query cost constant regardless of how deep the user scrolls.
