# Stored Procedures and User-Defined Functions

**Objective:** Encapsulate business logic using stored procedures and functions.

---

## Data

```
 id | customer_name | sale_date  | amount  |  product   
----+---------------+------------+---------+------------
  1 | Marcus Reid   | 2026-04-01 | 1200.00 | Laptop
  2 | Nina Patel    | 2026-04-03 |  450.00 | Phone
  3 | Omar Hassan   | 2026-04-05 |   89.00 | Headphones
  4 | Priya Nair    | 2026-04-07 | 2300.00 | Monitor
  5 | Rafael Costa  | 2026-04-08 |  340.00 | Keyboard
  6 | Sofia Mendes  | 2026-03-10 |  780.00 | Tablet
  7 | Tariq Malik   | 2026-03-15 |  560.00 | Phone
  8 | Yuna Choi     | 2026-03-20 | 1900.00 | Laptop
  9 | Zara Ahmed    | 2026-02-25 |  430.00 | Headphones
 10 | Liam Novak    | 2026-02-10 |  670.00 | Keyboard
(10 rows)
```

---

## Step 1: Stored Procedure — Total Sales by Date Range

Takes a `start_date` and `end_date`, validates that neither is NULL, then returns the total sales amount in that range via an `OUT` parameter.

```sql
CREATE OR REPLACE PROCEDURE total_sales
(
  start_date DATE,
  end_date DATE,
  OUT total DECIMAL
)
AS $$
BEGIN
  IF start_date IS NULL THEN
    RAISE EXCEPTION '`start` must have a value';
  END IF;
  IF end_date IS NULL THEN
    RAISE EXCEPTION '`end` must have a value';
  END IF;
  SELECT SUM(amount) INTO total
  FROM sales
  WHERE 
    sale_date >= start_date
    AND sale_date <= end_date;
END
$$ LANGUAGE plpgsql;
```

**Testing — NULL validation:**

```sql
CALL total_sales(NULL, NULL);
```
```
ERROR:  `start` must have a value
```

```sql
CALL total_sales('2026-03-01', NULL);
```
```
ERROR:  `end` must have a value
```

**Testing — valid call:**

```sql
CALL total_sales('2026-03-01', '2026-04-16', NULL);
```
```
  total  
---------
 7619.00
(1 row)
```

The `NULL` passed as the third argument is a placeholder for the `OUT` parameter — PostgreSQL requires it in the call signature.

---

## Step 2: Scalar Function — Discount Calculator

Takes a discount percentage and a price, returns the discounted price as a single value.

```sql
CREATE OR REPLACE FUNCTION calc_discount(discount INT, price INT)
RETURNS INT
AS $$
  DECLARE
    res INT;
  BEGIN
    res = price - (price * (discount / 100.0));
    RETURN res;
  END;
$$ LANGUAGE plpgsql;
```

**Test:**

```sql
SELECT calc_discount(10, 1000) AS discounted_price;
```
```
 discounted_price 
------------------
              900
(1 row)
```

---

## Step 3: Table-valued Function — Discount Applied to All Sales

Takes a discount percentage and returns a table with each sale's original and discounted amount.

```sql
CREATE OR REPLACE FUNCTION calc_discount_all(discount INT)
RETURNS TABLE(original DECIMAL, discounted DECIMAL)
AS $$
  BEGIN
    RETURN QUERY
    SELECT amount, (amount - (amount * (discount / 100.0)))
    FROM sales;
  END;
$$ LANGUAGE plpgsql;
```

**Test:**

```sql
SELECT * FROM calc_discount_all(10);
```
```
 original |         discounted          
----------+-----------------------------
  1200.00 | 1080.0000000000000000000000
   450.00 |  405.0000000000000000000000
    89.00 |   80.1000000000000000000000
  2300.00 | 2070.0000000000000000000000
   340.00 |  306.0000000000000000000000
   780.00 |  702.0000000000000000000000
   560.00 |  504.0000000000000000000000
  1900.00 | 1710.0000000000000000000000
   430.00 |  387.0000000000000000000000
   670.00 |  603.0000000000000000000000
(10 rows)
```
