# Date and Time Functions

**Objective:** Manipulate and query data based on date and time values.

---

## Data

```
 order_id |     name     | order_date | delivery_date |  status   | amount 
----------+--------------+------------+---------------+-----------+--------
        1 | Marcus Reid  | 2026-03-25 | 2026-03-30    | delivered | 120.00
        2 | Nina Patel   | 2026-03-28 | 2026-04-02    | delivered | 340.00
        3 | Omar Hassan  | 2026-04-01 | 2026-04-08    | shipped   |  89.50
        4 | Priya Nair   | 2026-04-05 | 2026-04-10    | delivered | 210.00
        5 | Rafael Costa | 2026-04-07 | 2026-04-14    | pending   | 175.00
        6 | Sofia Mendes | 2026-04-10 | 2026-04-18    | shipped   | 430.00
        7 | Tariq Malik  | 2026-02-10 | 2026-02-17    | delivered |  95.00
        8 | Yuna Choi    | 2026-01-15 | 2026-01-22    | pending   | 310.00
        9 | Zara Ahmed   | 2026-03-01 | 2026-03-08    | cancelled |  60.00
       10 | Liam Novak   | 2026-02-28 | 2026-03-07    | shipped   | 500.00
(10 rows)
```

---

## Step 1: Calculate Delivery Duration

Subtracting two `DATE` columns gives the number of days between them. PostgreSQL supports this directly with the `-` operator — other databases like MySQL use `DATEDIFF(delivery_date, order_date)` instead.

```sql
SELECT order_id, name, (delivery_date - order_date) AS duration
FROM orders;
```

**Output:**

```
 order_id |     name     | duration 
----------+--------------+----------
        1 | Marcus Reid  |        5
        2 | Nina Patel   |        5
        3 | Omar Hassan  |        7
        4 | Priya Nair   |        5
        5 | Rafael Costa |        7
        6 | Sofia Mendes |        8
        7 | Tariq Malik  |        7
        8 | Yuna Choi    |        7
        9 | Zara Ahmed   |        7
       10 | Liam Novak   |        7
(10 rows)
```

---

## Step 2: Filter by Date Range

Orders placed more than 30 days ago, using `CURRENT_DATE` to compute the difference dynamically.

```sql
SELECT order_id, name,
(CURRENT_DATE - order_date) AS order_placed_n_days_ago
FROM orders
WHERE (CURRENT_DATE - order_date) > 30;
```

**Output:**

```
 order_id |    name     | order_placed_n_days_ago 
----------+-------------+-------------------------
        7 | Tariq Malik |                      63
        8 | Yuna Choi   |                      89
        9 | Zara Ahmed  |                      44
       10 | Liam Novak  |                      45
(4 rows)
```

---

## Step 3: Format Date Output

By default, PostgreSQL renders dates as `YYYY-MM-DD`. `TO_CHAR` can reformat them into whatever display format is needed.

**Default:**

```sql
SELECT order_id, name, order_date
FROM orders;
```

```
 order_id |     name     | order_date 
----------+--------------+------------
        1 | Marcus Reid  | 2026-03-25
        2 | Nina Patel   | 2026-03-28
        3 | Omar Hassan  | 2026-04-01
        4 | Priya Nair   | 2026-04-05
        5 | Rafael Costa | 2026-04-07
        6 | Sofia Mendes | 2026-04-10
        7 | Tariq Malik  | 2026-02-10
        8 | Yuna Choi    | 2026-01-15
        9 | Zara Ahmed   | 2026-03-01
       10 | Liam Novak   | 2026-02-28
(10 rows)
```

**Formatted as DD-MM-YYYY:**

```sql
SELECT order_id, name, TO_CHAR(order_date, 'DD-MM-YYYY')
FROM orders;
```

```
 order_id |     name     |  to_char   
----------+--------------+------------
        1 | Marcus Reid  | 25-03-2026
        2 | Nina Patel   | 28-03-2026
        3 | Omar Hassan  | 01-04-2026
        4 | Priya Nair   | 05-04-2026
        5 | Rafael Costa | 07-04-2026
        6 | Sofia Mendes | 10-04-2026
        7 | Tariq Malik  | 10-02-2026
        8 | Yuna Choi    | 15-01-2026
        9 | Zara Ahmed   | 01-03-2026
       10 | Liam Novak   | 28-02-2026
(10 rows)
```
