# Comprehensive Database Design, Optimization, and Advanced Features

**Objective:** Design a normalized schema for an eCommerce platform and implement indexing, triggers, and transactions.

---

## Schema Design

The schema models a basic eCommerce platform with four tables: `products`, `customers`, `orders`, and `order_details`.

The separation of `orders` and `order_details` resolves a many-to-many relationship between `orders` and `products`. Storing `unit_price` in `order_details` captures the price at purchase — preventing changes in the product table from affecting historical orders.

The schema is in third normal form (3NF): no duplicate data, every non-key column depends only on its table's primary key, and relationships are expressed through foreign keys rather than repeated columns.

---

## Step 1: Table Creation

### products

```sql
CREATE TABLE products (
  product_id INT PRIMARY KEY,
  product_name VARCHAR(100) NOT NULL,
  price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
  stock INT NOT NULL DEFAULT 0 CHECK (stock >= 0)
);
```

`price` and `stock` both have `CHECK` constraints to prevent negative values. `stock` defaults to `0` so new products don't need an explicit stock value on insert.

### customers

```sql
CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  first_name VARCHAR(20) NOT NULL,
  last_name VARCHAR(20),
  email VARCHAR(50) UNIQUE NOT NULL
);
```

`email` is marked `UNIQUE NOT NULL` — it's the natural identifier for a customer and duplicates would cause data integrity issues.

### orders

```sql
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT NOT NULL REFERENCES customers(customer_id) ON DELETE RESTRICT,
  order_date DATE NOT NULL,
  total_amt DECIMAL(10, 2) NOT NULL CHECK (total_amt >= 0),
  status VARCHAR(20) NOT NULL CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);
```

`customer_id` is a foreign key to `customers` with `ON DELETE RESTRICT` — you can't delete a customer who has orders. The `status` column uses a `CHECK` constraint to act as a simple enum, rejecting anything outside the four valid values.

### order_details

```sql
CREATE TABLE order_details (
  order_details_id INT PRIMARY KEY,
  order_id INT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
  product_id INT NOT NULL REFERENCES products(product_id) ON DELETE RESTRICT,
  quantity INT CHECK (quantity > 0),
  unit_price DECIMAL(10, 2) NOT NULL
);
```

`order_id` uses `ON DELETE CASCADE` — if an order is deleted, its line items go with it. `product_id` uses `ON DELETE RESTRICT` — you can't delete a product that's been ordered. `quantity` must be greater than 0.

---

## Step 2: Constraint Testing

Before inserting real data, the constraints were verified by intentionally violating each one.

**Negative price:**
```sql
INSERT INTO products VALUES (1, 'Laptop', -100, 10);
```
```
ERROR:  new row for relation "products" violates check constraint "products_price_check"
```

**Duplicate email:**
```sql
INSERT INTO customers VALUES (1, 'Marcus', 'Reid', 'marcus@mail.com');
INSERT INTO customers VALUES (2, 'Nina', 'Patel', 'marcus@mail.com');
```
```
ERROR:  duplicate key value violates unique constraint "customers_email_key"
```

**Invalid status:**
```sql
INSERT INTO orders VALUES (1, 1, '2026-04-16', 500.00, 'processing');
```
```
ERROR:  new row for relation "orders" violates check constraint "orders_status_check"
```

**Zero quantity:**
```sql
INSERT INTO order_details VALUES (1, 1, 1, 0, 999.00);
```
```
ERROR:  new row for relation "order_details" violates check constraint "order_details_quantity_check"
```

**Non-existent foreign key:**
```sql
INSERT INTO orders VALUES (1, 999, '2026-04-16', 500.00, 'pending');
```
```
ERROR:  insert or update on table "orders" violates foreign key constraint "orders_customer_id_fkey"
```

All constraints behave as expected.

---

## Step 3: Data Insertion

```sql
INSERT INTO products (product_id, product_name, price, stock) VALUES
(1, 'Laptop', 999.99, 50),
(2, 'Phone', 499.99, 100),
(3, 'Monitor', 299.99, 30),
(4, 'Keyboard', 49.99, 200),
(5, 'Mouse', 29.99, 150),
(6, 'Headphones', 79.99, 80),
(7, 'Webcam', 59.99, 60),
(8, 'Tablet', 349.99, 40),
(9, 'Charger', 19.99, 300),
(10, 'USB Hub', 39.99, 120);

INSERT INTO customers (customer_id, first_name, last_name, email) VALUES
(1, 'Marcus', 'Reid', 'marcus@mail.com'),
(2, 'Nina', 'Patel', 'nina@mail.com'),
(3, 'Omar', 'Hassan', 'omar@mail.com'),
(4, 'Priya', 'Nair', 'priya@mail.com'),
(5, 'Rafael', 'Costa', 'rafael@mail.com'),
(6, 'Sofia', 'Mendes', 'sofia@mail.com'),
(7, 'Tariq', 'Malik', 'tariq@mail.com'),
(8, 'Yuna', 'Choi', 'yuna@mail.com'),
(9, 'Zara', 'Ahmed', 'zara@mail.com'),
(10, 'Liam', 'Novak', 'liam@mail.com');

INSERT INTO orders (order_id, customer_id, order_date, total_amt, status) VALUES
(1, 1, '2026-03-01', 999.99, 'delivered'),
(2, 2, '2026-03-05', 499.99, 'shipped'),
(3, 3, '2026-03-10', 299.99, 'pending'),
(4, 4, '2026-03-15', 149.98, 'delivered'),
(5, 5, '2026-03-20', 79.99, 'cancelled'),
(6, 6, '2026-03-25', 349.99, 'shipped'),
(7, 7, '2026-04-01', 59.99, 'pending'),
(8, 8, '2026-04-05', 1049.98, 'delivered'),
(9, 9, '2026-04-10', 19.99, 'shipped'),
(10, 10, '2026-04-14', 39.99, 'pending');

INSERT INTO order_details (order_details_id, order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 1, 999.99),
(2, 2, 2, 1, 499.99),
(3, 3, 3, 1, 299.99),
(4, 4, 4, 2, 49.99),
(5, 5, 6, 1, 79.99),
(6, 6, 8, 1, 349.99),
(7, 7, 7, 1, 59.99),
(8, 8, 1, 1, 999.99),
(9, 8, 5, 1, 49.99),
(10, 9, 9, 1, 19.99);
```

---

## Step 4: Indexing

Indexes speed up reads by letting the database locate rows without scanning the entire table. The tradeoff is slightly slower writes and extra storage — so indexes are added on columns that are frequently used in `WHERE`, `JOIN`, or `ORDER BY` clauses.

```sql
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
```
Speeds up lookups like "all orders for a customer" — a very common query pattern.

```sql
CREATE INDEX idx_orders_order_date ON orders(order_date);
```
Useful for date range filters, which are frequent in any reporting query.

```sql
CREATE INDEX idx_orders_status ON orders(status);
```
Supports filtering by order status, e.g. fetching all pending or shipped orders.

```sql
CREATE INDEX idx_order_details_product_id ON order_details(product_id);
```
Speeds up joins from `order_details` to `products`, and queries like "all orders containing product X".

```sql
CREATE INDEX idx_order_details_order_id ON order_details(order_id);
```
Speeds up fetching line items for a given order — this join happens on almost every order view.

```sql
CREATE INDEX idx_products_name ON products(product_name);
```
Supports product search by name.

```sql
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);
```
A composite index — covers queries that filter by both customer and date together, which is more efficient than two separate indexes for that case.

**All indexes:**

```
                           List of relations
 Schema |             Name             | Type  | Owner |     Table     
--------+------------------------------+-------+-------+---------------
 public | customers_email_key          | index | ayla  | customers
 public | customers_pkey               | index | ayla  | customers
 public | idx_order_details_order_id   | index | ayla  | order_details
 public | idx_order_details_product_id | index | ayla  | order_details
 public | idx_orders_customer_date     | index | ayla  | orders
 public | idx_orders_customer_id       | index | ayla  | orders
 public | idx_orders_order_date        | index | ayla  | orders
 public | idx_orders_status            | index | ayla  | orders
 public | idx_products_name            | index | ayla  | products
 public | order_details_pkey           | index | ayla  | orders
 public | orders_pkey                  | index | ayla  | orders
 public | products_pkey                | index | ayla  | products
(12 rows)
```

Note that `customers_pkey`, `orders_pkey`, etc. are created automatically by PostgreSQL for primary key columns. `customers_email_key` is auto-created for the `UNIQUE` constraint on email.

---

## Step 5: Triggers

### Trigger 1 — Deduct Stock on Order Insert

When a row is inserted into `order_details`, the stock for that product should automatically decrease by the quantity ordered. This is handled with a `BEFORE INSERT` trigger.

```sql
CREATE OR REPLACE FUNCTION set_updated_stock()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE products
  SET stock = stock - NEW.quantity
  WHERE product_id = NEW.product_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER trigger_set_updated_stock
BEFORE INSERT ON order_details
FOR EACH ROW
EXECUTE FUNCTION set_updated_stock();
```

`NEW` refers to the row being inserted. The trigger fires once per row, so inserting multiple line items in one statement deducts stock for each correctly.

### Trigger 2 — Log Price Changes

Any time a product's price is updated, the old and new values are recorded in a `log` table with a timestamp. This is an `AFTER UPDATE` trigger — it only fires if the price actually changed.

```sql
CREATE TABLE log (
  log_id SERIAL PRIMARY KEY,
  product_id INT NOT NULL REFERENCES products(product_id) ON DELETE CASCADE,
  old_price DECIMAL(10, 2) NOT NULL,
  new_price DECIMAL(10, 2) NOT NULL,
  changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION log_price_change()
RETURNS TRIGGER AS $$
BEGIN
  IF OLD.price <> NEW.price THEN
    INSERT INTO log (product_id, old_price, new_price)
    VALUES (NEW.product_id, OLD.price, NEW.price);
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE TRIGGER trigger_log_price_change
AFTER UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION log_price_change();
```

`OLD` and `NEW` are both available in update triggers — `OLD` is the row before the update, `NEW` is the row after. The `IF OLD.price <> NEW.price` check means the log only gets a row if the price actually changed, not on every update to the products table.

---

## Step 6: Transactions

A transaction groups multiple operations into a single unit — either everything commits, or nothing does. This is critical when operations are dependent on each other.

### COMMIT — Stock deduction confirmed

```sql
BEGIN;
  INSERT INTO order_details (order_details_id, order_id, product_id, quantity, unit_price)
  VALUES (11, 10, 2, 2, 499.99);

  SELECT product_id, stock FROM products WHERE product_id = 2;
```
```
 product_id | stock 
------------+-------
          2 |    98
```
```sql
COMMIT;

SELECT product_id, stock FROM products WHERE product_id = 2;
```
```
 product_id | stock 
------------+-------
          2 |    98
```

Stock dropped from 100 to 98 (quantity 2 deducted by the trigger), and the change persisted after `COMMIT`.

### ROLLBACK — Stock deduction reversed

```sql
BEGIN;
  INSERT INTO order_details (order_details_id, order_id, product_id, quantity, unit_price)
  VALUES (12, 10, 3, 1, 299.99);

  SELECT product_id, stock FROM products WHERE product_id = 3;
```
```
 product_id | stock 
------------+-------
          3 |    29
```
```sql
ROLLBACK;

SELECT product_id, stock FROM products WHERE product_id = 3;
```
```
 product_id | stock 
------------+-------
          3 |    30
```

Stock temporarily dropped to 29 inside the transaction, but `ROLLBACK` undid everything — the trigger's stock deduction included — and the stock returned to 30.

### Price change log within a transaction

```sql
BEGIN;
  UPDATE products SET price = 599.99 WHERE product_id = 2;

  SELECT * FROM log WHERE product_id = 2;
```
```
 log_id | product_id | old_price | new_price |         changed_at         
--------+------------+-----------+-----------+----------------------------
      1 |          2 |    499.99 |    599.99 | 2026-04-16 18:14:28.061943
```
```sql
COMMIT;

SELECT product_id, price FROM products WHERE product_id = 2;
```
```
 product_id | price  
------------+--------
          2 | 599.99
```

The price change trigger fired within the transaction and logged the old and new price. After `COMMIT`, both the price update and the log entry are permanently saved.
