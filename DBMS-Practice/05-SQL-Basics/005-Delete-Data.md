# Delete Data

## Scenario
Someone on your team ran `DELETE FROM sessions;` in production, forgetting the `WHERE` clause meant to remove only expired sessions - wiping every active user session.

## Problem
1. Write the correct delete statement that removes only expired sessions.
2. Suggest two safeguards that would have prevented the accident.

## Solution
```sql
DELETE FROM sessions
WHERE expires_at < now();
```

Safeguards:
1. **Soft deletes**: instead of `DELETE`, add a `deleted_at TIMESTAMPTZ` column and `UPDATE ... SET deleted_at = now()`; actual physical deletion happens later via a separate, reviewed cleanup job.
2. **Require WHERE by policy**: many teams configure their SQL client/CI to reject `DELETE`/`UPDATE` statements without a `WHERE` clause (e.g. MySQL's "safe update mode", or a linter in CI for migration files), and require destructive prod queries to go through a reviewed runbook, not an ad-hoc terminal session.

## Takeaway
`DELETE` without `WHERE` removes every row in the table - always run the equivalent `SELECT COUNT(*) ... WHERE ...` first, prefer soft deletes for anything user-facing, and never run raw destructive SQL directly against production without review.
