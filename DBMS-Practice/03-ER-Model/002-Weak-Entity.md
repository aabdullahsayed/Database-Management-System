# Weak Entity

## Scenario
Continuing the ride-share app: an `Invoice Line Item` only makes sense in the context of a specific `Invoice`. It has no meaningful identity on its own - `line_number` is only unique *within* an invoice.

## Diagram
```
+-------------+          1        M       +---------------------+
|  Invoice    |========================>>>|  Invoice Line Item   |  (weak entity, double-bordered)
+-------------+     "has"                  +---------------------+
| invoice_id PK|                            | invoice_id PK,FK    |
| customer_id  |                            | line_number PK      |
| created_at   |                            | description         |
+-------------+                             | amount              |
                                             +---------------------+
```

## Problem
Model `Invoice Line Item` as a weak entity dependent on `Invoice`. What must its primary key look like?

## Solution
A weak entity has no standalone primary key - it borrows the primary key of its owning (identifying) entity and combines it with a **partial key** (discriminator) to form its own composite primary key.

```sql
CREATE TABLE InvoiceLineItem (
    invoice_id   INT NOT NULL,
    line_number  INT NOT NULL,
    description  VARCHAR(200),
    amount       DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (invoice_id, line_number),
    FOREIGN KEY (invoice_id) REFERENCES Invoice(invoice_id) ON DELETE CASCADE
);
```

## Takeaway
Weak entities' primary keys are always composite: `(owner's PK, partial key)`, and they should cascade-delete when the owning strong entity is deleted, since they can't exist without it.
