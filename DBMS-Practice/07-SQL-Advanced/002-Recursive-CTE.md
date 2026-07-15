# Recursive CTE

## Scenario
You have a `categories` table with `parent_id` (unlimited depth: Electronics -> Computers -> Laptops -> Gaming Laptops), and need to fetch a category and *all* its descendants, no matter how deep.

## Solution
```sql
WITH RECURSIVE category_tree AS (
    -- anchor: the starting category
    SELECT id, name, parent_id, 0 AS depth
    FROM categories
    WHERE id = 5   -- e.g. "Electronics"

    UNION ALL

    -- recursive step: find children of what we've found so far
    SELECT c.id, c.name, c.parent_id, ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY depth;
```

The anchor member runs once to seed the result; the recursive member repeatedly joins against the growing `category_tree` result set until no new rows are produced (no more children found).

## Takeaway
A self-join (from the earlier lesson) only walks one level of a hierarchy; a recursive CTE walks arbitrary depth - this is the standard SQL tool for trees and graphs (org charts, category trees, "reports-to" chains, bill-of-materials explosions).
