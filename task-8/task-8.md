# Task 08 — Common Table Expressions (CTEs) and Recursive Queries

**Objective:** Simplify complex queries and process hierarchical data using CTEs.

---

## Data

**staff**
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

**org**
```
 id |        name         | manager_id 
----+---------------------+------------
  1 | CEO                 |           
  2 | VP Engineering      |          1
  3 | VP Finance          |          1
  4 | Engineering Manager |          2
  5 | Finance Manager     |          3
  6 | Engineer A          |          4
  7 | Engineer B          |          4
  8 | Accountant          |          5
(8 rows)
```

---

## Step 1: Non-recursive CTE

Two chained CTEs — the first computes the department average, the second joins it back to `staff` and filters employees above that average.

```sql
WITH cte AS (
  SELECT department, avg(salary) dept_avg
  FROM staff
  GROUP BY department
),

filter_cte AS (
  SELECT s.id, s.name, s.department, s.salary, ROUND(c.dept_avg, 2)
  FROM staff s
  JOIN cte c ON s.department = c.department
  WHERE s.salary > c.dept_avg
)

SELECT * FROM filter_cte;
```

**Output:**

```
 id |     name     | department  |  salary   |  round   
----+--------------+-------------+-----------+----------
  1 | Marcus Reid  | Engineering |  95000.00 | 92500.00
  4 | Priya Nair   | Engineering | 105000.00 | 92500.00
  5 | Rafael Costa | Finance     |  72000.00 | 70666.67
  7 | Tariq Malik  | Finance     |  72000.00 | 70666.67
 10 | Liam Novak   | Product     |  91000.00 | 82333.33
(5 rows)
```

---

## Step 2: Recursive CTE — Org Chart

A recursive CTE has two parts joined by `UNION ALL` — a base case and a recursive case. The base case runs once and returns the starting rows (here, the CEO where `manager_id IS NULL`). The recursive case keeps joining against the previous result until no new rows are produced.


Traverses the `org` table hierarchy starting from the CEO down to individual contributors.

```sql
WITH RECURSIVE org_tree AS (
  -- base case
  SELECT id, name, manager_id, 1 AS level
  FROM org WHERE manager_id IS NULL
  UNION ALL

  -- recursive
  SELECT o.id, o.name, o.manager_id, ot.level + 1
  FROM org o
  JOIN org_tree ot ON o.manager_id = ot.id
)

SELECT * FROM org_tree ORDER BY level;
```

**Output:**

```
 id |        name         | manager_id | level 
----+---------------------+------------+-------
  1 | CEO                 |            |     1
  2 | VP Engineering      |          1 |     2
  3 | VP Finance          |          1 |     2
  4 | Engineering Manager |          2 |     3
  5 | Finance Manager     |          3 |     3
  6 | Engineer A          |          4 |     4
  7 | Engineer B          |          4 |     4
  8 | Accountant          |          5 |     4
(8 rows)
```

---

## Step 3: Termination of Recursive CTEs

Termination is guaranteed as long as the data doesn't contain cycles — i.e., no row is its own ancestor. In this org chart, every node eventually traces back to the CEO and stops there. If there were a cycle (say, row A's manager is B and B's manager is A), the recursion would never terminate.
