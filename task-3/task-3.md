# Aggregate Functions and Grouping

**Objective:** Summarize data using aggregate functions and grouping.

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

## Step 1: Count Employees per Department

```sql
SELECT department, COUNT(*) AS employee_count
FROM employee
GROUP BY department;
```

**Output:**

```
 department  | employee_count 
-------------+----------------
 Product     |              2
 Finance     |              3
 Engineering |              4
 HR          |              1
(4 rows)
```

---

## Step 2: Average Salary per Department

```sql
SELECT department, AVG(salary) AS avg_employee_salary
FROM employee
GROUP BY department;
```

**Output:**

```
 department  | avg_employee_salary 
-------------+---------------------
 Product     |  77000.000000000000
 Finance     |  72000.000000000000
 Engineering |  92000.000000000000
 HR          |  95000.000000000000
(4 rows)
```

---

## Step 3: Total Salary per Role and Department

```sql
SELECT role, department, SUM(salary) AS avg_salary_per_role_department
FROM employee
GROUP BY role, department;
```

**Output:**

```
   role   | department  | avg_salary_per_role_department 
----------+-------------+--------------------------------
 Designer | Product     |                      154000.00
 Analyst  | Finance     |                      216000.00
 Manager  | Engineering |                      105000.00
 Manager  | HR          |                       95000.00
 Engineer | Engineering |                      263000.00
(5 rows)
```

---

## Step 4: Departments with Average Salary Above 80,000 (HAVING)

```sql
SELECT department, AVG(salary) AS avg_employee_salary
FROM employee
GROUP BY department
HAVING AVG(salary) > 80000;
```

**Output:**

```
 department  | avg_employee_salary 
-------------+---------------------
 Engineering |  92000.000000000000
 HR          |  95000.000000000000
(2 rows)
```
