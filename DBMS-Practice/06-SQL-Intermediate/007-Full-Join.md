# Full Join

## Scenario
You're reconciling two systems during a migration: `legacy_users` and `new_users`. You need every user that exists in *either* system, flagging which side(s) they appear on, to find missing/duplicate migrations.

## Solution
```sql
SELECT
    COALESCE(legacy.email, new_sys.email) AS email,
    legacy.id  IS NOT NULL AS in_legacy,
    new_sys.id IS NOT NULL AS in_new_system
FROM legacy_users legacy
FULL JOIN new_users new_sys ON legacy.email = new_sys.email;
```

`FULL JOIN` (a.k.a. `FULL OUTER JOIN`) keeps unmatched rows from **both** sides, NULL-filling whichever side lacks a match - so a user only in `legacy_users` shows `in_legacy = true, in_new_system = false`, and vice versa. Rows in neither... don't exist, by definition (at least one side is always non-NULL).

## Takeaway
`FULL JOIN` is the tool for reconciliation/diffing tasks between two datasets - "show me everything from both sides, matched where possible, flagged where not."
