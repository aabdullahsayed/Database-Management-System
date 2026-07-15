# Triggers

## Scenario
Compliance requires an immutable audit log: every time a row in the `salaries` table is updated, the old value must automatically be recorded, even if a developer forgets to add logging code in the application.

## Solution
```sql
CREATE TABLE salary_audit (
    id SERIAL PRIMARY KEY,
    employee_id INT,
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    changed_at TIMESTAMPTZ DEFAULT now()
);

CREATE FUNCTION log_salary_change() RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO salary_audit(employee_id, old_salary, new_salary)
    VALUES (OLD.id, OLD.salary, NEW.salary);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER salary_update_trigger
AFTER UPDATE OF salary ON salaries
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary)
EXECUTE FUNCTION log_salary_change();
```

The trigger fires automatically on every `UPDATE` that changes `salary`, regardless of which application, script, or admin console performed the update - there's no way to bypass it short of disabling the trigger itself (which is itself an auditable DDL action).

## Takeaway
Triggers enforce behavior (auditing, validation, denormalized cache updates) at the database level so it can't be accidentally skipped by any one application code path - but use them sparingly, since hidden trigger logic can make debugging "why did this row change" much harder if overused.
