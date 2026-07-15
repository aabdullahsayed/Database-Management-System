# Strong Entity

## Scenario
You're modeling a ride-sharing app. A `Driver` can exist and be identified on its own (has a `driver_id`). Contrast this with a `Vehicle Inspection Record` that only makes sense tied to a specific vehicle.

## Diagram
```
+-------------+
|   Driver    |   <- strong entity: has its own primary key (driver_id)
+-------------+
| driver_id PK|
| name        |
| license_no  |
+-------------+
```

## Problem
Define a strong entity and give the schema for `Driver` as a strong entity, including its key.

## Solution
A **strong entity** has its own primary key and exists independently in the database - its existence doesn't depend on any other entity.

```sql
CREATE TABLE Driver (
    driver_id   INT PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    license_no  VARCHAR(30) UNIQUE NOT NULL
);
```

`Driver` rows are meaningful and identifiable on their own, independent of any `Vehicle` or `Trip`.

## Takeaway
If an entity has a natural, independent primary key and its rows make sense without referencing another entity, model it as a strong entity with a simple `CREATE TABLE ... PRIMARY KEY`.
