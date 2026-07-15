# Create Database

## Scenario
You're spinning up a new microservice ("inventory-service") and need its own isolated database on the shared Postgres instance so it doesn't collide with other teams' tables.

## Problem
Write the statement to create a new database `inventory_service`, and explain why services should typically get their own database/schema rather than sharing one.

## Solution
```sql
CREATE DATABASE inventory_service
    WITH ENCODING 'UTF8'
    OWNER = inventory_svc_user;
```

Isolating databases per service:
- Prevents naming collisions (two teams both wanting a `users` table).
- Lets you set per-service access control (only `inventory_svc_user` can touch this DB).
- Makes it possible to back up, scale, or migrate one service's data independently of others.

## Takeaway
`CREATE DATABASE` is a namespace/isolation boundary, not just "a place to put tables" - in microservice architectures, database-per-service is a common practice to avoid tight coupling through shared tables.
