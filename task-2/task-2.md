# Basic Filtering and Sorting

**Objective:** Filter records using `WHERE` and sort results using `ORDER BY`, including multi-condition queries.

---

## Data

```
 id | first_name | last_name |     email      |   role   | hire_date  | department  |  salary   
----+------------+-----------+----------------+----------+------------+-------------+-----------
  1 | Alice      | Johnson   | alice@corp.com | Engineer | 2021-03-15 | Engineering |  85000.00
  2 | Bob        | Smith     | bob@corp.com   | Manager  | 2019-07-01 | HR          |  95000.00
  3 | Carol      | White     | carol@corp.com | Analyst  | 2022-01-10 | Finance     |  72000.00
  4 | David      | Brown     | david@corp.com | Engineer | 2020-11-23 | Engineering |  88000.00
  5 | Eva        | Davis     | eva@corp.com   | Designer | 2023-02-14 | Product     |  78000.00
  6 | Frank      | Miller    | frank@corp.com | Analyst  | 2021-09-05 | Finance     |  71000.00
  7 | Grace      | Wilson    | grace@corp.com | Engineer | 2022-06-18 | Engineering |  90000.00
  8 | Henry      | Moore     | henry@corp.com | Manager  | 2018-04-30 | Engineering | 105000.00
  9 | Iris       | Taylor    | iris@corp.com  | Designer | 2023-08-01 | Product     |  76000.00
 10 | Jack       | Anderson  | jack@corp.com  | Analyst  | 2020-12-12 | Finance     |  73000.00
(10 rows)
```

---

## Step 1: Filter by Department

```sql
SELECT first_name, last_name, email, department
FROM employee
WHERE department = 'Finance';
```

**Output:**

```
 first_name | last_name |     email      | department 
------------+-----------+----------------+------------
 Carol      | White     | carol@corp.com | Finance
 Frank      | Miller    | frank@corp.com | Finance
 Jack       | Anderson  | jack@corp.com  | Finance
(3 rows)
```

---

## Step 2: Sort by Salary and Email

```sql
SELECT * FROM employee 
ORDER BY salary DESC, email ASC;
```

**Output:**

```
 id | first_name | last_name |     email      |   role   | hire_date  | department  |  salary   
----+------------+-----------+----------------+----------+------------+-------------+-----------
  8 | Henry      | Moore     | henry@corp.com | Manager  | 2018-04-30 | Engineering | 105000.00
  2 | Bob        | Smith     | bob@corp.com   | Manager  | 2019-07-01 | HR          |  95000.00
  7 | Grace      | Wilson    | grace@corp.com | Engineer | 2022-06-18 | Engineering |  90000.00
  4 | David      | Brown     | david@corp.com | Engineer | 2020-11-23 | Engineering |  88000.00
  1 | Alice      | Johnson   | alice@corp.com | Engineer | 2021-03-15 | Engineering |  85000.00
  5 | Eva        | Davis     | eva@corp.com   | Designer | 2023-02-14 | Product     |  78000.00
  9 | Iris       | Taylor    | iris@corp.com  | Designer | 2023-08-01 | Product     |  76000.00
 10 | Jack       | Anderson  | jack@corp.com  | Analyst  | 2020-12-12 | Finance     |  73000.00
  3 | Carol      | White     | carol@corp.com | Analyst  | 2022-01-10 | Finance     |  72000.00
  6 | Frank      | Miller    | frank@corp.com | Analyst  | 2021-09-05 | Finance     |  71000.00
(10 rows)
```

---

## Step 3: Multiple Conditions with AND + IN

```sql
SELECT * FROM employee
WHERE department IN ('Engineering', 'Finance')
AND salary > 80000;
```

**Output:**

```
 id | first_name | last_name |     email      |   role   | hire_date  | department  |  salary   
----+------------+-----------+----------------+----------+------------+-------------+-----------
  1 | Alice      | Johnson   | alice@corp.com | Engineer | 2021-03-15 | Engineering |  85000.00
  4 | David      | Brown     | david@corp.com | Engineer | 2020-11-23 | Engineering |  88000.00
  7 | Grace      | Wilson    | grace@corp.com | Engineer | 2022-06-18 | Engineering |  90000.00
  8 | Henry      | Moore     | henry@corp.com | Manager  | 2018-04-30 | Engineering | 105000.00
(4 rows)
```

---

## Step 4: Filtering by Role and Hire Date

```sql
SELECT * FROM employee
WHERE role IN ('Engineer', 'Analyst')
AND hire_date > '2020-01-01';
```

**Output:**

```
 id | first_name | last_name |     email      |   role   | hire_date  | department  |  salary  
----+------------+-----------+----------------+----------+------------+-------------+----------
  1 | Alice      | Johnson   | alice@corp.com | Engineer | 2021-03-15 | Engineering | 85000.00
  3 | Carol      | White     | carol@corp.com | Analyst  | 2022-01-10 | Finance     | 72000.00
  4 | David      | Brown     | david@corp.com | Engineer | 2020-11-23 | Engineering | 88000.00
  6 | Frank      | Miller    | frank@corp.com | Analyst  | 2021-09-05 | Finance     | 71000.00
  7 | Grace      | Wilson    | grace@corp.com | Engineer | 2022-06-18 | Engineering | 90000.00
 10 | Jack       | Anderson  | jack@corp.com  | Analyst  | 2020-12-12 | Finance     | 73000.00
(6 rows)
```
