# Window Functions and Ranking

**Objective:** Leverage window functions to perform calculations across a set of rows.

---

## Data

```
 id |     name     | department  |  salary   | hire_date  
----+--------------+-------------+-----------+------------
  1 | Marcus Reid  | Engineering |  95000.00 | 2021-03-15
  2 | Nina Patel   | Engineering |  85000.00 | 2022-06-10
  3 | Omar Hassan  | Engineering |  85000.00 | 2020-11-23
  4 | Priya Nair   | Engineering | 105000.00 | 2018-04-30
  5 | Rafael Costa | Finance     |  72000.00 | 2021-09-05
  6 | Sofia Mendes | Finance     |  68000.00 | 2022-01-10
  7 | Tariq Malik  | Finance     |  72000.00 | 2020-12-12
  8 | Yuna Choi    | Product     |  78000.00 | 2023-02-14
  9 | Zara Ahmed   | Product     |  78000.00 | 2023-08-01
 10 | Liam Novak   | Product     |  91000.00 | 2019-07-01
(10 rows)
```

---

## Step 1: ROW_NUMBER() — Company-wide Salary Ranking

Ranks all employees by salary across the entire table. No ties — every row gets a unique number.

```sql
SELECT name, salary,
ROW_NUMBER() OVER(ORDER BY salary DESC) AS salary_ranking_company
FROM staff;
```

**Output:**

```
     name     |  salary   | salary_ranking_company 
--------------+-----------+------------------------
 Priya Nair   | 105000.00 |                      1
 Marcus Reid  |  95000.00 |                      2
 Liam Novak   |  91000.00 |                      3
 Nina Patel   |  85000.00 |                      4
 Omar Hassan  |  85000.00 |                      5
 Yuna Choi    |  78000.00 |                      6
 Zara Ahmed   |  78000.00 |                      7
 Tariq Malik  |  72000.00 |                      8
 Rafael Costa |  72000.00 |                      9
 Sofia Mendes |  68000.00 |                     10
(10 rows)
```

---

## Step 2: ROW_NUMBER vs RANK vs DENSE_RANK — Per Department

`PARTITION BY` resets the window per department. The three functions handle ties differently:

- `ROW_NUMBER` — always unique, arbitrary order for ties
- `RANK` — ties get the same rank, next rank skips (1, 1, 3)
- `DENSE_RANK` — ties get the same rank, next rank doesn't skip (1, 1, 2)

```sql
SELECT name, salary,
ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) AS row_number,
RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS rank,
DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS dense_rank
FROM staff;
```

**Output:**

```
     name     |  salary   | row_number | rank | dense_rank 
--------------+-----------+------------+------+------------
 Priya Nair   | 105000.00 |          1 |    1 |          1
 Marcus Reid  |  95000.00 |          2 |    2 |          2
 Nina Patel   |  85000.00 |          3 |    3 |          3
 Omar Hassan  |  85000.00 |          4 |    3 |          3
 Tariq Malik  |  72000.00 |          1 |    1 |          1
 Rafael Costa |  72000.00 |          2 |    1 |          1
 Sofia Mendes |  68000.00 |          3 |    3 |          2
 Liam Novak   |  91000.00 |          1 |    1 |          1
 Yuna Choi    |  78000.00 |          2 |    2 |          2
 Zara Ahmed   |  78000.00 |          3 |    2 |          2
(10 rows)
```

Finance is a good example — Rafael and Tariq are tied at 72000. `RANK` jumps from 1 to 3, skipping 2. `DENSE_RANK` goes 1, 1, 2 without the gap.

---

## Step 3: LEAD() and LAG()

`LEAD` looks at the next row's value, `LAG` looks at the previous one — both relative to the current row's position in the window. Useful for comparing adjacent values. The first row has no previous (`LAG` = NULL), the last has no next (`LEAD` = NULL).

```sql
SELECT name, salary,
LEAD(salary) OVER(ORDER BY salary) AS lead_salary,
LAG(salary) OVER(ORDER BY salary) AS lag_salary
FROM staff;
```

**Output:**

```
     name     |  salary   | lead_salary | lag_salary 
--------------+-----------+-------------+------------
 Sofia Mendes |  68000.00 |   72000.00  |           
 Tariq Malik  |  72000.00 |   72000.00  |  68000.00
 Rafael Costa |  72000.00 |   78000.00  |  72000.00
 Zara Ahmed   |  78000.00 |   78000.00  |  72000.00
 Yuna Choi    |  78000.00 |   85000.00  |  78000.00
 Omar Hassan  |  85000.00 |   85000.00  |  78000.00
 Nina Patel   |  85000.00 |   91000.00  |  85000.00
 Liam Novak   |  91000.00 |   95000.00  |  85000.00
 Marcus Reid  |  95000.00 | 105000.00   |  91000.00
 Priya Nair   | 105000.00 |             |  95000.00
(10 rows)
```
