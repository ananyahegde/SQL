# Multi-Table JOINs

**Objective:** Combine data from two related tables using JOIN operations.

---

## Tables

**customers**
```
 customer_id | first_name | last_name |     email      
-------------+------------+-----------+----------------
           1 | Alice      | Johnson   | alice@mail.com
           2 | Bob        | Smith     | bob@mail.com
           3 | Carol      | White     | carol@mail.com
           4 | David      | Brown     | david@mail.com
           5 | Eva        | Davis     | eva@mail.com
           6 | Frank      | Miller    | frank@mail.com
           7 | Grace      | Wilson    | grace@mail.com
           8 | Henry      | Moore     | henry@mail.com
           9 | Iris       | Taylor    | iris@mail.com
          10 | Jack       | Anderson  | jack@mail.com
          11 | Leo        | King      | leo@mail.com
          12 | Mia        | Fox       | mia@mail.com
(12 rows)
```

**orders**
```
 order_id | customer_id | order_date |  status   | amount 
----------+-------------+------------+-----------+--------
        1 |           1 | 2024-01-05 | shipped   |    250
        2 |           2 | 2024-01-08 | pending   |    120
        3 |           3 | 2024-02-11 | delivered |    340
        4 |           4 | 2024-02-20 | shipped   |     80
        5 |           5 | 2024-03-01 | cancelled |    210
        6 |           6 | 2024-03-15 | delivered |     95
        7 |           7 | 2024-04-02 | pending   |    430
        8 |           8 | 2024-04-18 | shipped   |    175
        9 |           9 | 2024-05-05 | delivered |    310
       10 |          10 | 2024-06-01 | pending   |     60
       11 |           1 | 2024-07-01 | delivered |    500
       12 |           1 | 2024-08-15 | pending   |    300
       13 |           3 | 2024-09-10 | shipped   |    150
(13 rows)
```

---
## Step 1: Creating two tables `customers` and `orders` with foreign key relationship
 
Two tables with a foreign key relationship:
 
```sql
CREATE TABLE customers (
  customer_id SERIAL PRIMARY KEY,
  first_name VARCHAR(20),
  last_name VARCHAR(20),
  email VARCHAR(20)
);
 
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  customer_id INT REFERENCES customers(customer_id),
  order_date DATE,
  status VARCHAR(10),
  amount INT
);
```
---

## Step 2: INNER JOIN

`INNER JOIN` return only rows where there's a match in both tables. Customers with no orders won't show up. <br>
Note: Using `JOIN` defaults to inner join.


```sql
SELECT c.first_name, c.last_name, o.order_id, o.order_date, o.status, o.amount
FROM customers c JOIN orders o
ON c.customer_id = o.customer_id;
```

**Output:**

```
 first_name | last_name | order_id | order_date |  status   | amount 
------------+-----------+----------+------------+-----------+--------
 Alice      | Johnson   |        1 | 2024-01-05 | shipped   |    250
 Bob        | Smith     |        2 | 2024-01-08 | pending   |    120
 Carol      | White     |        3 | 2024-02-11 | delivered |    340
 David      | Brown     |        4 | 2024-02-20 | shipped   |     80
 Eva        | Davis     |        5 | 2024-03-01 | cancelled |    210
 Frank      | Miller    |        6 | 2024-03-15 | delivered |     95
 Grace      | Wilson    |        7 | 2024-04-02 | pending   |    430
 Henry      | Moore     |        8 | 2024-04-18 | shipped   |    175
 Iris       | Taylor    |        9 | 2024-05-05 | delivered |    310
 Jack       | Anderson  |       10 | 2024-06-01 | pending   |     60
 Alice      | Johnson   |       11 | 2024-07-01 | delivered |    500
 Alice      | Johnson   |       12 | 2024-08-15 | pending   |    300
 Carol      | White     |       13 | 2024-09-10 | shipped   |    150
(13 rows)
```

Customer with id 12 and 13 do not show up in the inner join since there is no entry of them in `orders` table.

---

## Step 3: LEFT JOIN

Returns all customers, including those with no orders. Missing order fields come back as `NULL`.

```sql
SELECT c.customer_id, c.first_name, c.last_name, COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

**Output:**

```
 customer_id | first_name | last_name | order_count 
-------------+------------+-----------+-------------
           1 | Alice      | Johnson   |           3
           2 | Bob        | Smith     |           1
           3 | Carol      | White     |           2
           4 | David      | Brown     |           1
           5 | Eva        | Davis     |           1
           6 | Frank      | Miller    |           1
           7 | Grace      | Wilson    |           1
           8 | Henry      | Moore     |           1
           9 | Iris       | Taylor    |           1
          10 | Jack       | Anderson  |           1
          11 | Leo        | King      |           0
          12 | Mia        | Fox       |           0
(12 rows)
```

Leo and Mia show up with `order_count = 0` — they'd be dropped entirely with an INNER JOIN.
