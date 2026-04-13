# Creating and Populating Tables

**Objective:** Set up a table, insert data, and verify it with a basic query.

---

## Step 1: Create the Database

```bash
createdb -U <username> task-1
```

---

## Step 2: Connect to the Database

```bash
psql -h localhost -p 5432 -d task-1 -U <username> -W
```

Enter your password when prompted. You should land in the `task-1=#` shell.

---

## Step 3: Create the Table

```sql
CREATE TABLE employee (
  id SERIAL PRIMARY KEY,
  first_name VARCHAR(20) NOT NULL,
  last_name VARCHAR(20) NOT NULL,
  email VARCHAR(15) UNIQUE NOT NULL,
  role VARCHAR(15),
  hire_date DATE NOT NULL,
  department VARCHAR(30),
  salary DECIMAL(10, 2)
);
```

---

## Step 4: Insert Data

```sql
INSERT INTO employee (first_name, last_name, email, role, hire_date, department, salary) VALUES
('Alice', 'Johnson', 'alice@corp.com', 'Engineer', '2021-03-15', 'Engineering', 85000.00),
('Bob', 'Smith', 'bob@corp.com', 'Manager', '2019-07-01', 'HR', 95000.00),
('Carol', 'White', 'carol@corp.com', 'Analyst', '2022-01-10', 'Finance', 72000.00),
('David', 'Brown', 'david@corp.com', 'Engineer', '2020-11-23', 'Engineering', 88000.00),
('Eva', 'Davis', 'eva@corp.com', 'Designer', '2023-02-14', 'Product', 78000.00),
('Frank', 'Miller', 'frank@corp.com', 'Analyst', '2021-09-05', 'Finance', 71000.00),
('Grace', 'Wilson', 'grace@corp.com', 'Engineer', '2022-06-18', 'Engineering', 90000.00),
('Henry', 'Moore', 'henry@corp.com', 'Manager', '2018-04-30', 'Engineering', 105000.00),
('Iris', 'Taylor', 'iris@corp.com', 'Designer', '2023-08-01', 'Product', 76000.00),
('Jack', 'Anderson', 'jack@corp.com', 'Analyst', '2020-12-12', 'Finance', 73000.00);
```

---

## Step 5: Verify the Data

```sql
SELECT * FROM employee;
```

**Output:**

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

All 10 rows returned — insertion verified.
