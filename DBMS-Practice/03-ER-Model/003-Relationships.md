# Relationships

## Scenario
Product wants to add: "a customer can review a product, and each review has a rating and text." You need to model the relationship between `Customer` and `Product`.

## Diagram
```
+----------+          M        M        +----------+
| Customer |=========================>>>| Product  |
+----------+        "reviews"           +----------+
     \                                        /
      \                                      /
       \        Review (rating, text)       /
        \______________________________ ___/
```

## Problem
Since a review has its own attributes (rating, text, created_at) beyond just linking a customer to a product, how should this relationship be modeled as a relational table?

## Solution
A many-to-many relationship with its own attributes becomes its own table (an "associative" or "junction" table) holding both foreign keys plus the relationship's attributes:

```sql
CREATE TABLE Review (
    customer_id INT NOT NULL,
    product_id  INT NOT NULL,
    rating      SMALLINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    text        TEXT,
    created_at  TIMESTAMP NOT NULL DEFAULT now(),
    PRIMARY KEY (customer_id, product_id),
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id),
    FOREIGN KEY (product_id)  REFERENCES Product(product_id)
);
```

## Takeaway
Relationships between entities become their own tables when they are many-to-many and/or carry their own attributes; the relationship table's primary key is usually the combination of the participating entities' foreign keys.
