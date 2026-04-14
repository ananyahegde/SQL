# Subqueries and Nested Queries

**Objective:** Use subqueries to filter or compute values within a main query.

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

## Step 1: Subquery in WHERE (Correlated)

Fetch employees who earn above their own department's average — the subquery runs once per row, referencing the outer query's department.

```sql
SELECT e.first_name, e.last_name, e.department, e.salary
FROM employee e
WHERE salary > (SELECT AVG(salary)
                FROM employee se
                WHERE se.department = e.department);
```

**Output:**

```
 first_name | last_name | department  |  salary   
------------+-----------+-------------+-----------
 Eva        | Davis     | Product     |  78000.00
 Henry      | Moore     | Engineering | 105000.00
 Jack       | Anderson  | Finance     |  73000.00
(3 rows)
```

---

## Step 2: Subquery in SELECT

### Non-correlated — company-wide average

The subquery runs once and returns the same value for every row.

```sql
SELECT first_name, last_name, 
department,
(SELECT AVG(salary) FROM employee) AS company_avg_salary,
salary
FROM employee;
```

**Output:**

```
 first_name | last_name | department  | company_avg_salary |  salary   
------------+-----------+-------------+--------------------+-----------
 Alice      | Johnson   | Engineering | 83300.000000000000 |  85000.00
 Bob        | Smith     | HR          | 83300.000000000000 |  95000.00
 Carol      | White     | Finance     | 83300.000000000000 |  72000.00
 David      | Brown     | Engineering | 83300.000000000000 |  88000.00
 Eva        | Davis     | Product     | 83300.000000000000 |  78000.00
 Frank      | Miller    | Finance     | 83300.000000000000 |  71000.00
 Grace      | Wilson    | Engineering | 83300.000000000000 |  90000.00
 Henry      | Moore     | Engineering | 83300.000000000000 | 105000.00
 Iris       | Taylor    | Product     | 83300.000000000000 |  76000.00
 Jack       | Anderson  | Finance     | 83300.000000000000 |  73000.00
(10 rows)
```

### Correlated — department average per row

The subquery runs once per row, scoped to that employee's department.

```sql
SELECT e.first_name, e.last_name, 
e.department,
(SELECT AVG(salary) 
 FROM employee se 
 WHERE e.department = se.department) AS company_avg_salary,
e.salary
FROM employee e;
```

**Output:**

```
 first_name | last_name | department  | company_avg_salary |  salary   
------------+-----------+-------------+--------------------+-----------
 Alice      | Johnson   | Engineering | 92000.000000000000 |  85000.00
 Bob        | Smith     | HR          | 95000.000000000000 |  95000.00
 Carol      | White     | Finance     | 72000.000000000000 |  72000.00
 David      | Brown     | Engineering | 92000.000000000000 |  88000.00
 Eva        | Davis     | Product     | 77000.000000000000 |  78000.00
 Frank      | Miller    | Finance     | 72000.000000000000 |  71000.00
 Grace      | Wilson    | Engineering | 92000.000000000000 |  90000.00
 Henry      | Moore     | Engineering | 92000.000000000000 | 105000.00
 Iris       | Taylor    | Product     | 77000.000000000000 |  76000.00
 Jack       | Anderson  | Finance     | 72000.000000000000 |  73000.00
(10 rows)
```

---

## Step 3: Correlated vs Non-correlated

A **non-correlated** subquery runs independently — it doesn't reference anything from the outer query, so the database executes it once and reuses the result.

A **correlated** subquery references a column from the outer query, so it re-executes for every row the outer query processes.

To map this to what's above:
- **Step 1** — WHERE with a correlated subquery (department average recalculated per row)
- **Step 2** — SELECT with both: first query is non-correlated (same company average repeated), second is correlated (department average per row)
- Below — WHERE with a non-correlated subquery for contrast:

```sql
SELECT first_name, last_name, department, salary
FROM employee
WHERE salary > (SELECT AVG(salary)
                FROM employee);
```

**Output:**

```
 first_name | last_name | department  |  salary   
------------+-----------+-------------+-----------
 Alice      | Johnson   | Engineering |  85000.00
 Bob        | Smith     | HR          |  95000.00
 David      | Brown     | Engineering |  88000.00
 Grace      | Wilson    | Engineering |  90000.00
 Henry      | Moore     | Engineering | 105000.00
(5 rows)
```

The subquery here computes the overall average once — anyone above that threshold gets returned.
