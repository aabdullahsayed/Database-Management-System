# Design an Ecommerce Database

## Scenario
An e-commerce platform needs products (with variants like size/color), a shopping cart, orders that snapshot product info at purchase time (so a later price change doesn't retroactively alter past invoices), and inventory tracking per variant.

## Requirements
- A product can have multiple variants (e.g. T-shirt: size x color combinations), each with its own price/stock.
- Orders must preserve historical price/name at time of purchase, even if the product later changes.
- Cart contents are per-user and can be abandoned (not every cart becomes an order).

## Diagram
```
Product        ProductVariant         Cart          CartItem         Order        OrderItem
+--------+  1 M +--------------+     +------+  1  M +---------+     +-------+  1 M +-----------+
|prod_id |----->| variant_id   |     |cart  |------>|cart_item|     |order  |----->|order_item |
|name    |      | product_id FK|     |_id   |       |variant  |     |_id    |      |variant_id |
+--------+      | sku          |     +------+       |_id FK   |     |total  |      |name_snap  |  <- snapshot!
                | price        |                     +---------+     +-------+      |price_snap |  <- snapshot!
                | stock        |                                                    +-----------+
                +--------------+
```

## Schema
```sql
CREATE TABLE Product (
    product_id SERIAL PRIMARY KEY,
    name       VARCHAR(200) NOT NULL
);

CREATE TABLE ProductVariant (
    variant_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL REFERENCES Product(product_id),
    sku        VARCHAR(50) UNIQUE NOT NULL,
    price      DECIMAL(10,2) NOT NULL,
    stock      INT NOT NULL DEFAULT 0 CHECK (stock >= 0)
);

CREATE TABLE Cart (
    cart_id    SERIAL PRIMARY KEY,
    user_id    INT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE CartItem (
    cart_id    INT NOT NULL REFERENCES Cart(cart_id),
    variant_id INT NOT NULL REFERENCES ProductVariant(variant_id),
    quantity   INT NOT NULL CHECK (quantity > 0),
    PRIMARY KEY (cart_id, variant_id)
);

CREATE TABLE "Order" (
    order_id   SERIAL PRIMARY KEY,
    user_id    INT NOT NULL,
    total      DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE OrderItem (
    order_id       INT NOT NULL REFERENCES "Order"(order_id),
    variant_id     INT NOT NULL REFERENCES ProductVariant(variant_id),
    name_snapshot  VARCHAR(200) NOT NULL,   -- name AT TIME OF PURCHASE
    price_snapshot DECIMAL(10,2) NOT NULL,  -- price AT TIME OF PURCHASE
    quantity       INT NOT NULL CHECK (quantity > 0),
    PRIMARY KEY (order_id, variant_id)
);
```

## Key design decisions
- `OrderItem` deliberately duplicates `name_snapshot`/`price_snapshot` instead of just joining live to `ProductVariant` - this is intentional denormalization: a past invoice must never change just because someone edited the product catalog later. This is a case where normalization would actively be *wrong*.
- `CartItem` has NO snapshot columns - carts are transient/live, so they should always reflect current price/name; only the moment of purchase (`OrderItem`) needs to freeze that data.
- `stock` uses `CHECK (stock >= 0)` plus, in the checkout transaction, a `SELECT ... FOR UPDATE` or atomic `UPDATE ... WHERE stock >= quantity` to prevent overselling under concurrent checkouts (see Concurrency-Control/Locks).

## Takeaway
Not everything should be normalized - order line items are the textbook case for intentional denormalization (snapshotting), because "what the customer was charged" must be immutable history, independent of the live, ever-changing product catalog.
