# SQL Cheat Sheet

> A practical, GitHub-ready reference for **SQL syntax, descriptions, and examples**, including **MySQL**, **IBM Db2 (LUW)**, **SQLite**, and Python database workflows.
>
> **How to read this guide:** Replace placeholders such as `<table_name>`, `<column_name>`, and `<condition>` with your own values. Square brackets in *syntax templates* indicate optional parts; do not type the brackets literally. SQL examples are formatted for clarity and may require the example tables to exist first.

## Table of Contents

1. [SQL overview and example tables](#1-sql-overview-and-example-tables)
2. [Retrieving data (SELECT / DQL)](#2-retrieving-data-select--dql)
3. [Changing data (INSERT, UPDATE, DELETE / DML)](#3-changing-data-insert-update-delete--dml)
4. [Defining and modifying tables (DDL)](#4-defining-and-modifying-tables-ddl)
5. [Aggregate, numeric, and string functions](#5-aggregate-numeric-and-string-functions)
6. [Date and time functions](#6-date-and-time-functions)
7. [Subqueries](#7-subqueries)
8. [Joins](#8-joins)
9. [SQLite with Python (`sqlite3`)](#9-sqlite-with-python-sqlite3)
10. [Pandas, CSV files, and visualizations](#10-pandas-csv-files-and-visualizations)
11. [IBM Db2 with Python (`ibm_db`)](#11-ibm-db2-with-python-ibm_db)
12. [Dialect differences and common mistakes](#12-dialect-differences-and-common-mistakes)
13. [Official references](#13-official-references)

---

## 1. SQL overview and example tables

### Main command families

| Family | Purpose | Commands |
| --- | --- | --- |
| **DQL** (Data Query Language) | Retrieve data | `SELECT` |
| **DML** (Data Manipulation Language) | Add, edit, or remove rows | `INSERT`, `UPDATE`, `DELETE` |
| **DDL** (Data Definition Language) | Create or change database structures | `CREATE`, `ALTER`, `TRUNCATE`, `DROP` |
| **TCL** (Transaction Control Language) | Manage transactions | `COMMIT`, `ROLLBACK` |

### Example tables referenced throughout

These are **illustrative table/column names**, not a complete schema. Use the names in your actual database when running the examples.

| Table | Example columns |
| --- | --- |
| `placeofinterest` | `name`, `type`, `city`, `country`, `airport` |
| `employees` | `emp_id`, `f_name`, `l_name`, `salary`, `dep_id`, `job_id`, `mobile`, `b_date`, `manager_id` |
| `jobs` | `job_ident`, `job_title` |
| `departments` | `dept_id_dep`, `dep_name` |
| `locations` | `loct_id`, `city` |
| `job_history` | `empl_id`, `start_date` |
| `instructor` | `id`, `first_name`, `last_name`, `city`, `country` |
| `international_student_test_scores` | `country`, `first_name`, `last_name`, `test_score` |

**Conventions:** Use `'single quotes'` for SQL string literals. `*` means *all columns* in `SELECT`, but *all rows* inside `COUNT(*)`. SQL keywords are uppercase, though SQL keywords are generally case-insensitive.

---

## 2. Retrieving data (SELECT / DQL)

### 2.1 `SELECT` — Retrieve columns

**Description:** Fetch data from a table. Specify columns or use `*` to return all columns.

**Syntax**
```sql
SELECT <column_1>, <column_2>, ...
FROM <table_name>;
```

**Example**
```sql
SELECT city
FROM placeofinterest;
```

### 2.2 `WHERE` — Filter rows

**Description:** Include only rows satisfying a condition.

**Syntax**
```sql
SELECT <column_1>, <column_2>, ...
FROM <table_name>
WHERE <condition>;
```

**Example**
```sql
SELECT *
FROM placeofinterest
WHERE city = 'Rome';
```

### 2.3 `DISTINCT` — Return unique combinations

**Description:** Eliminate duplicate rows from the **selected columns**. `DISTINCT` is a keyword, not a function.

**Syntax**
```sql
SELECT DISTINCT <column_1> [, <column_2>, ...]
FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
SELECT DISTINCT country
FROM placeofinterest
WHERE type = 'historical';
```

### 2.4 `LIMIT` / `FETCH FIRST` — Restrict result size

**Description:** Return at most a specified number of rows. Use `ORDER BY` to make the chosen rows predictable.

**Syntax — MySQL / SQLite**
```sql
SELECT <columns>
FROM <table_name>
[ORDER BY <column_name> ASC|DESC]
LIMIT <number_of_rows>;
```

**Example — MySQL / SQLite**
```sql
SELECT *
FROM placeofinterest
WHERE airport = 'Pearson'
ORDER BY name
LIMIT 5;
```

**Syntax — Db2**
```sql
SELECT <columns>
FROM <table_name>
[ORDER BY <column_name> ASC|DESC]
FETCH FIRST <number_of_rows> ROWS ONLY;
```

**Example — Db2**
```sql
SELECT *
FROM placeofinterest
ORDER BY name
FETCH FIRST 5 ROWS ONLY;
```

### 2.5 `ORDER BY` — Sort results

**Description:** Sort ascending (`ASC`, default) or descending (`DESC`).

**Syntax**
```sql
SELECT <columns>
FROM <table_name>
ORDER BY <column_1> [ASC|DESC], <column_2> [ASC|DESC];
```

**Example**
```sql
SELECT emp_id, f_name, salary
FROM employees
ORDER BY salary DESC, f_name ASC;
```

### 2.6 Common `WHERE` operators

**Description:** Combine conditions or test sets, ranges, patterns, and missing values.

**Syntax**
```sql
WHERE <column_name> = <value>                 -- equals
WHERE <column_name> <> <value>                -- not equals
WHERE <condition_1> AND <condition_2>         -- both true
WHERE <condition_1> OR <condition_2>          -- either true
WHERE <column_name> IN (<value_1>, <value_2>) -- belongs to list
WHERE <column_name> BETWEEN <low> AND <high>  -- inclusive range
WHERE <column_name> LIKE <pattern>            -- pattern matching
WHERE <column_name> IS NULL                  -- missing value
WHERE <column_name> IS NOT NULL              -- present value
```

**Examples**
```sql
SELECT *
FROM employees
WHERE salary BETWEEN 40000 AND 80000
  AND dep_id IN (2, 5);

SELECT *
FROM placeofinterest
WHERE name LIKE 'N%';  -- starts with N

SELECT *
FROM employees
WHERE mobile IS NULL;
```

**Remember:** `%` in `LIKE` matches any sequence of characters; `_` matches one character. Use `IS NULL`, not `= NULL`.

### 2.7 `GROUP BY` — Aggregate by category

**Description:** Group rows together before applying aggregate functions.

**Syntax**
```sql
SELECT <group_column>, <aggregate_function>(<value_column>) AS <alias>
FROM <table_name>
[WHERE <row_condition>]
GROUP BY <group_column>;
```

**Example**
```sql
SELECT dep_id, COUNT(*) AS employee_count
FROM employees
GROUP BY dep_id;
```

### 2.8 `HAVING` — Filter grouped results

**Description:** Apply conditions to groups **after aggregation**. `WHERE` filters individual rows first.

**Syntax**
```sql
SELECT <group_column>, COUNT(*) AS <alias>
FROM <table_name>
GROUP BY <group_column>
HAVING COUNT(*) <comparison_operator> <value>;
```

**Example**
```sql
SELECT dep_id, COUNT(*) AS employee_count
FROM employees
GROUP BY dep_id
HAVING COUNT(*) >= 5;
```

---

## 3. Changing data (INSERT, UPDATE, DELETE / DML)

### 3.1 `INSERT INTO` — Add rows

**Description:** Insert one or more new records.

**Syntax**
```sql
INSERT INTO <table_name> (<column_1>, <column_2>, ...)
VALUES (<value_1>, <value_2>, ...);
```

**Example**
```sql
INSERT INTO placeofinterest (name, type, city, country, airport)
VALUES ('Niagara Waterfalls', 'Nature', 'Toronto', 'Canada', 'Pearson');
```

### 3.2 `UPDATE` — Change existing rows

**Description:** Modify matching rows. Without `WHERE`, **every row may be updated**.

**Syntax**
```sql
UPDATE <table_name>
SET <column_1> = <new_value_1>,
    <column_2> = <new_value_2>
[WHERE <condition>];
```

**Example**
```sql
UPDATE placeofinterest
SET name = 'Niagara Falls'
WHERE name = 'Niagara Waterfalls';
```

### 3.3 `DELETE FROM` — Remove matching rows

**Description:** Delete records that meet a condition. Without `WHERE`, **every row may be deleted**.

**Syntax**
```sql
DELETE FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
DELETE FROM placeofinterest
WHERE city IN ('Rome', 'Vienna');
```

### 3.4 `COMMIT` and `ROLLBACK` — Manage transactions

**Description:** Save a transaction or undo uncommitted changes. Exact transaction behavior depends on DBMS and driver settings.

**Syntax**
```sql
COMMIT;
ROLLBACK;
```

**Example (a transaction already started)**
```sql
UPDATE employees
SET salary = salary * 1.05
WHERE dep_id = 5;

-- Choose one, depending on whether you want to keep the change:
COMMIT;
-- ROLLBACK;
```

**Safety tip:** Run a matching `SELECT ... WHERE ...` before an `UPDATE` or `DELETE`, and test destructive statements inside a transaction where your database supports it.

---

## 4. Defining and modifying tables (DDL)

### 4.1 `CREATE TABLE` — Define a new table

**Description:** Create columns with data types and constraints such as `PRIMARY KEY` and `NOT NULL`.

**Syntax**
```sql
CREATE TABLE <table_name> (
    <column_1> <data_type> [<constraints>],
    <column_2> <data_type> [<constraints>],
    ...
);
```

**Example — MySQL / Db2**
```sql
CREATE TABLE employee (
    employee_id CHAR(5) PRIMARY KEY,
    first_name VARCHAR(30) NOT NULL,
    mobile VARCHAR(20)
);
```

**Additional example — SQLite: international student test scores**
```sql
CREATE TABLE international_student_test_scores (
    country VARCHAR(50),
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    test_score INTEGER
);
```

**Note:** Store phone numbers as text, not `INT` (they may contain leading zeros, `+`, or punctuation). In SQLite, `VARCHAR(50)` does **not** enforce a 50-character limit by itself.

### 4.2 `ALTER TABLE ... ADD COLUMN` — Add a column

**Description:** Extend an existing table.

**Syntax — MySQL / Db2**
```sql
ALTER TABLE <table_name>
ADD COLUMN <new_column_name> <data_type> [<constraints>];
```

**Example**
```sql
ALTER TABLE employee
ADD COLUMN income BIGINT;
```

**Note:** `COLUMN` is optional in MySQL and many other systems. For multiple new columns, repeat `ADD COLUMN` in MySQL; Db2 and SQLite have their own restrictions, so a separate `ALTER TABLE` per column is a portable approach.

### 4.3 `ALTER TABLE ... MODIFY / ALTER COLUMN` — Change a data type

**Description:** Modify a column definition. The keyword differs by database; constraints and existing values may limit allowed changes.

**Syntax — MySQL**
```sql
ALTER TABLE <table_name>
MODIFY COLUMN <column_name> <new_data_type> [<constraints>];
```

**Example — MySQL**
```sql
ALTER TABLE employee
MODIFY COLUMN mobile VARCHAR(25);
```

**Syntax — Db2**
```sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> SET DATA TYPE <new_data_type>;
```

**Example — Db2**
```sql
ALTER TABLE employee
ALTER COLUMN mobile SET DATA TYPE VARCHAR(25);
```

**SQLite:** Does not offer a general `MODIFY COLUMN` operation; many type changes require rebuilding the table.

### 4.4 `ALTER TABLE ... DROP COLUMN` — Remove a column

**Description:** Permanently remove a column and its stored values (subject to DBMS restrictions).

**Syntax**
```sql
ALTER TABLE <table_name>
DROP COLUMN <column_name>;
```

**Example**
```sql
ALTER TABLE employee
DROP COLUMN mobile;
```

### 4.5 `ALTER TABLE ... RENAME COLUMN` — Rename a column

**Description:** Change the identifier used for an existing column.

**Syntax — MySQL 8+ / Db2 / SQLite 3.25+**
```sql
ALTER TABLE <table_name>
RENAME COLUMN <old_column_name> TO <new_column_name>;
```

**Example**
```sql
ALTER TABLE employee
RENAME COLUMN first_name TO name;
```

**Alternative MySQL syntax (`CHANGE COLUMN`)**
```sql
-- Template: supply the full new column definition.
ALTER TABLE <table_name>
CHANGE COLUMN <old_column_name> <new_column_name> <data_type>;

-- Example:
ALTER TABLE employee
CHANGE COLUMN first_name name VARCHAR(30) NOT NULL;
```

### 4.6 `TRUNCATE TABLE` — Remove every row

**Description:** Empty a table while retaining its definition. It differs from `DELETE` in transaction, identity-reset, trigger, and logging behavior across engines.

**Syntax — MySQL**
```sql
TRUNCATE TABLE <table_name>;
```

**Example — MySQL**
```sql
TRUNCATE TABLE employee;
```

**Syntax — Db2**
```sql
TRUNCATE TABLE <table_name> IMMEDIATE;
```

**Example — Db2**
```sql
TRUNCATE TABLE employee IMMEDIATE;
```

**SQLite:** No `TRUNCATE TABLE`; use `DELETE FROM <table_name>;`. **Warning:** Db2 `IMMEDIATE` truncation cannot be rolled back, and MySQL `TRUNCATE` usually causes an implicit commit.

### 4.7 `DROP TABLE` — Delete a table

**Description:** Remove the table definition and its data.

**Syntax**
```sql
DROP TABLE <table_name>;
```

**Example**
```sql
DROP TABLE employee;
```

**Optional defensive variant (MySQL / SQLite; also supported in many systems)**
```sql
DROP TABLE IF EXISTS employee;
```

---

## 5. Aggregate, numeric, and string functions

### 5.1 `COUNT()` — Count rows or non-null values

**Description:** `COUNT(*)` counts rows; `COUNT(column)` counts **non-NULL** entries; `COUNT(DISTINCT column)` counts distinct non-NULL values.

**Syntax**
```sql
SELECT COUNT(*) AS <alias>
FROM <table_name>
[WHERE <condition>];

SELECT COUNT(<column_name>) AS <alias>
FROM <table_name>
[WHERE <condition>];
```

**Examples**
```sql
SELECT COUNT(*) AS total_places
FROM placeofinterest;

SELECT COUNT(country) AS canadian_places
FROM placeofinterest
WHERE country = 'Canada';

SELECT COUNT(DISTINCT dep_id) AS department_count
FROM employees;
```

### 5.2 `AVG()` — Arithmetic mean

**Description:** Find the average of non-NULL numeric values.

**Syntax**
```sql
SELECT AVG(<numeric_column>) AS <alias>
FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

### 5.3 `SUM()` — Total

**Description:** Add non-NULL numeric values.

**Syntax**
```sql
SELECT SUM(<numeric_column>) AS <alias>
FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
SELECT SUM(salary) AS total_salary
FROM employees;
```

### 5.4 `MIN()` — Smallest value

**Description:** Find the minimum non-NULL value.

**Syntax**
```sql
SELECT MIN(<column_name>) AS <alias>
FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
SELECT MIN(salary) AS lowest_salary
FROM employees;
```

### 5.5 `MAX()` — Largest value

**Description:** Find the maximum non-NULL value.

**Syntax**
```sql
SELECT MAX(<column_name>) AS <alias>
FROM <table_name>
[WHERE <condition>];
```

**Example**
```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

### 5.6 `ROUND()` — Round a number

**Description:** Round a numeric expression to a requested number of decimal places (default is often zero).

**Syntax**
```sql
SELECT ROUND(<numeric_expression>, <decimal_places>) AS <alias>
FROM <table_name>;
```

**Example**
```sql
SELECT emp_id, ROUND(salary, 2) AS rounded_salary
FROM employees;
```

**Correction:** `ROUND(2number, decimals, operation)` is not the usual MySQL/Db2 syntax; the third `operation` argument should be removed.

### 5.7 `LENGTH()` — Measure string length

**Description:** Get string length; **units differ by DBMS**. MySQL `LENGTH` counts bytes, whereas SQLite `length()` counts Unicode characters for text.

**Syntax**
```sql
SELECT LENGTH(<text_column>) AS <alias>
FROM <table_name>;
```

**Example**
```sql
SELECT f_name, LENGTH(f_name) AS name_length
FROM employees;
```

**MySQL character-count alternative:** `CHAR_LENGTH(f_name)`.

### 5.8 `UCASE()` / `UPPER()` — Convert text to uppercase

**Description:** Return the text value in uppercase; it does **not** rename the column. `UPPER()` is the portable spelling, and `UCASE()` is also supported in MySQL and Db2.

**Syntax**
```sql
SELECT UPPER(<text_column>) AS <alias>
FROM <table_name>;
```

**Example**
```sql
SELECT UPPER(f_name) AS uppercase_name
FROM employees;
```

### 5.9 `LCASE()` / `LOWER()` — Convert text to lowercase

**Description:** Return text in lowercase; `LOWER()` is the portable spelling.

**Syntax**
```sql
SELECT LOWER(<text_column>) AS <alias>
FROM <table_name>;
```

**Example**
```sql
SELECT LOWER(f_name) AS lowercase_name
FROM employees;
```

### 5.10 Combining `DISTINCT` with functions

**Description:** Apply a function first, then eliminate duplicate results.

**Syntax**
```sql
SELECT DISTINCT <function>(<column_name>)
FROM <table_name>;
```

**Example**
```sql
SELECT DISTINCT UPPER(f_name) AS unique_names
FROM employees;
```

---

## 6. Date and time functions

> **Important:** Date functions are **not fully portable**. Every subsection labels its supported dialects or provides equivalents.

### 6.1 `DAY()` — Day of the month (MySQL / Db2)

**Description:** Extract the day number (`1`–`31`) from a date.

**Syntax**
```sql
SELECT DAY(<date_column>) AS <alias>
FROM <table_name>;
```

**Example**
```sql
SELECT DAY(b_date) AS birth_day
FROM employees
WHERE emp_id = 'E1002';
```

**SQLite equivalent**
```sql
SELECT CAST(STRFTIME('%d', b_date) AS INTEGER) AS birth_day
FROM employees
WHERE emp_id = 'E1002';
```

### 6.2 `CURRENT_DATE` — Today's date

**Description:** Return the database's current date (timezone and session settings can affect the date).

**Syntax**
```sql
SELECT CURRENT_DATE;
```

**Example**
```sql
SELECT CURRENT_DATE AS today;
```

### 6.3 `DATEDIFF()` — Days between two dates (MySQL)

**Description:** Return `date_1 - date_2` in days, ignoring time-of-day components.

**Syntax — MySQL**
```sql
SELECT DATEDIFF(<date_1>, <date_2>) AS <alias>;
```

**Example — MySQL**
```sql
SELECT DATEDIFF(CURRENT_DATE, b_date) AS days_since_birth
FROM employees;
```

**Db2 equivalent**
```sql
SELECT DAYS(CURRENT DATE) - DAYS(b_date) AS days_since_birth
FROM employees;
```

**SQLite equivalent**
```sql
SELECT CAST(JULIANDAY('now', 'start of day') - JULIANDAY(b_date) AS INTEGER)
       AS days_since_birth
FROM employees;
```

**Note:** SQLite date strings should use a supported format such as `YYYY-MM-DD`. These calculations count elapsed **days**, not age in years.

### 6.4 `FROM_DAYS()` — Convert a MySQL day number to a date

**Description:** Convert MySQL's day number (as used by `TO_DAYS()`) back into a date. **MySQL-specific.**

**Syntax — MySQL**
```sql
SELECT FROM_DAYS(<day_number>) AS <alias>;
```

**Example — MySQL**
```sql
SELECT FROM_DAYS(TO_DAYS('2026-11-19')) AS restored_date;
```

**Important correction:** `FROM_DAYS(DATEDIFF(CURRENT_DATE, some_date))` **does not** convert elapsed days into years of age or into a duration; it interprets that number as a calendar day number. For age in years, use `TIMESTAMPDIFF(YEAR, b_date, CURRENT_DATE)` in MySQL.

### 6.5 `DATE_ADD()` — Add a date interval (MySQL)

**Description:** Add a specified number of days, months, or years to a date.

**Syntax — MySQL**
```sql
SELECT DATE_ADD(<date_expression>, INTERVAL <n> <DAY|MONTH|YEAR>) AS <alias>;
```

**Example — MySQL**
```sql
SELECT DATE_ADD('2026-11-19', INTERVAL 3 DAY) AS future_date;
```

**Db2 equivalent**
```sql
SELECT DATE('2026-11-19') + 3 DAYS AS future_date
FROM SYSIBM.SYSDUMMY1;
```

**SQLite equivalent**
```sql
SELECT DATE('2026-11-19', '+3 days') AS future_date;
```

### 6.6 `DATE_SUB()` — Subtract a date interval (MySQL)

**Description:** Subtract a specified number of days, months, or years from a date.

**Syntax — MySQL**
```sql
SELECT DATE_SUB(<date_expression>, INTERVAL <n> <DAY|MONTH|YEAR>) AS <alias>;
```

**Example — MySQL**
```sql
SELECT DATE_SUB('2026-11-19', INTERVAL 3 DAY) AS past_date;
```

**Db2 equivalent**
```sql
SELECT DATE('2026-11-19') - 3 DAYS AS past_date
FROM SYSIBM.SYSDUMMY1;
```

**SQLite equivalent**
```sql
SELECT DATE('2026-11-19', '-3 days') AS past_date;
```

---

## 7. Subqueries

**Description:** A subquery is a `SELECT` nested inside another SQL statement. It can appear in `WHERE`, `FROM`, `SELECT`, or other valid positions—not only in `WHERE`.

### 7.1 Scalar subquery in `WHERE`

**Syntax**
```sql
SELECT <columns>
FROM <outer_table>
WHERE <column_name> <comparison_operator> (
    SELECT <aggregate_function>(<column_name>)
    FROM <inner_table>
);
```

**Example: employees earning below average salary**
```sql
SELECT emp_id, f_name, l_name, salary
FROM employees
WHERE salary < (
    SELECT AVG(salary)
    FROM employees
);
```

### 7.2 Derived table (subquery in `FROM`)

**Syntax**
```sql
SELECT <columns>
FROM (
    SELECT <columns>
    FROM <table_name>
) AS <subquery_alias>;
```

**Example**
```sql
SELECT *
FROM (
    SELECT emp_id, f_name, l_name, dep_id
    FROM employees
) AS emp4all;
```

### 7.3 `IN` subquery

**Syntax**
```sql
SELECT <columns>
FROM <outer_table>
WHERE <column_name> IN (
    SELECT <column_name>
    FROM <inner_table>
);
```

**Example: employees with a matching job**
```sql
SELECT *
FROM employees
WHERE job_id IN (
    SELECT job_ident
    FROM jobs
);
```

---

## 8. Joins

**Description:** Joins combine records from multiple tables. Prefer **explicit `JOIN ... ON ...` syntax** for readability and to reduce accidental Cartesian products.

### Join types at a glance

| Join | Which rows are returned? |
| --- | --- |
| `INNER JOIN` | Only matching rows from both tables |
| `LEFT [OUTER] JOIN` | All left rows; matching right rows, or `NULL` |
| `RIGHT [OUTER] JOIN` | All right rows; matching left rows, or `NULL` |
| `FULL [OUTER] JOIN` | All rows from either side; unmatched side becomes `NULL` |
| `CROSS JOIN` | Every left row paired with every right row |
| Self join | A table joined to itself using different aliases |

### 8.1 Implicit inner join (older style)

**Description:** Put tables in a comma-separated `FROM` list and match them in `WHERE`.

**Syntax**
```sql
SELECT <columns>
FROM <table_1>, <table_2>
WHERE <table_1>.<join_column> = <table_2>.<join_column>;
```

**Example**
```sql
SELECT e.emp_id, e.f_name, j.job_title
FROM employees AS e, jobs AS j
WHERE e.job_id = j.job_ident;
```

**Preferred:** Use the explicit `INNER JOIN` in section 8.3.

### 8.2 `CROSS JOIN` / implicit cross join

**Description:** Produce every possible pair. If the left table has `m` rows and right has `n`, the result has `m × n` rows (before filtering).

**Syntax — explicit**
```sql
SELECT <columns>
FROM <table_1>
CROSS JOIN <table_2>;
```

**Syntax — implicit**
```sql
SELECT <columns>
FROM <table_1>, <table_2>;
```

**Examples**
```sql
SELECT d.dept_id_dep, l.loct_id
FROM departments AS d
CROSS JOIN locations AS l;

-- Implicit equivalent (avoid for clarity):
SELECT *
FROM employees, jobs;
```

### 8.3 `INNER JOIN` — Only matching rows

**Syntax**
```sql
SELECT <columns>
FROM <table_1> AS <alias_1>
INNER JOIN <table_2> AS <alias_2>
    ON <alias_1>.<join_column> = <alias_2>.<join_column>
[WHERE <condition>];
```

**Example**
```sql
SELECT e.f_name, e.l_name, jh.start_date
FROM employees AS e
INNER JOIN job_history AS jh
    ON e.emp_id = jh.empl_id
WHERE e.dep_id = 5;
```

### 8.4 `LEFT OUTER JOIN` — Keep all left rows

**Syntax**
```sql
SELECT <columns>
FROM <table_1> AS <alias_1>
LEFT OUTER JOIN <table_2> AS <alias_2>
    ON <alias_1>.<join_column> = <alias_2>.<join_column>
[WHERE <condition>];
```

**Example**
```sql
SELECT e.emp_id, e.l_name, e.dep_id, d.dep_name
FROM employees AS e
LEFT OUTER JOIN departments AS d
    ON e.dep_id = d.dept_id_dep;
```

**Tip:** A `WHERE d.dep_name = 'Sales'` filter eliminates unmatched right rows; put right-table filtering conditions in `ON` if you want to preserve unmatched left rows.

### 8.5 `RIGHT OUTER JOIN` — Keep all right rows

**Syntax**
```sql
SELECT <columns>
FROM <table_1> AS <alias_1>
RIGHT OUTER JOIN <table_2> AS <alias_2>
    ON <alias_1>.<join_column> = <alias_2>.<join_column>
[WHERE <condition>];
```

**Example**
```sql
SELECT e.emp_id, e.l_name, e.dep_id, d.dep_name
FROM employees AS e
RIGHT OUTER JOIN departments AS d
    ON e.dep_id = d.dept_id_dep;
```

**Tip:** Swapping table order and using `LEFT JOIN` often makes queries easier to read.

### 8.6 `FULL OUTER JOIN` — Keep rows from both sides

**Description:** Return matched pairs plus employees with no department and departments with no employees.

**Syntax — Db2 / modern SQLite**
```sql
SELECT <columns>
FROM <table_1> AS <alias_1>
FULL OUTER JOIN <table_2> AS <alias_2>
    ON <alias_1>.<join_column> = <alias_2>.<join_column>;
```

**Example — Db2 / SQLite 3.39+**
```sql
SELECT e.f_name, e.l_name, d.dep_name
FROM employees AS e
FULL OUTER JOIN departments AS d
    ON e.dep_id = d.dept_id_dep;
```

**MySQL does not implement `FULL OUTER JOIN` directly; use the workaround below.**

### 8.7 Full outer join workaround (MySQL)

**Description:** Combine a left join (matched + left-only rows) with **only unmatched right-side rows**. `UNION ALL` keeps duplicate records correctly.

**Syntax**
```sql
SELECT <columns>
FROM <left_table> AS l
LEFT JOIN <right_table> AS r
    ON l.<join_key> = r.<join_key>

UNION ALL

SELECT <same_columns_in_same_order>
FROM <left_table> AS l
RIGHT JOIN <right_table> AS r
    ON l.<join_key> = r.<join_key>
WHERE l.<non_nullable_primary_key> IS NULL;
```

**Example — MySQL**
```sql
SELECT e.emp_id, e.l_name, d.dep_name
FROM employees AS e
LEFT JOIN departments AS d
    ON e.dep_id = d.dept_id_dep

UNION ALL

SELECT e.emp_id, e.l_name, d.dep_name
FROM employees AS e
RIGHT JOIN departments AS d
    ON e.dep_id = d.dept_id_dep
WHERE e.emp_id IS NULL;
```

**Why not blindly use `UNION`?** `UNION` removes duplicate *result rows*, potentially deleting legitimate duplicates. Filtering the second branch for unmatched rows plus `UNION ALL` avoids that issue. The tested left-side identifier should be non-NULL for a genuine match (ideally a primary key).

### 8.8 Self join — Compare rows within one table

**Description:** Join a table to itself using two aliases. This example finds coworkers sharing a manager with employee `E1001`.

**Syntax**
```sql
SELECT <columns>
FROM <table_name> AS a
JOIN <table_name> AS b
    ON a.<relationship_column> = b.<relationship_column>
WHERE <condition>;
```

**Example**
```sql
SELECT b.emp_id, b.f_name, b.l_name
FROM employees AS a
INNER JOIN employees AS b
    ON a.manager_id = b.manager_id
WHERE a.emp_id = 'E1001'
  AND b.emp_id <> a.emp_id;
```

---

## 9. SQLite with Python (`sqlite3`)

> `sqlite3` is part of Python's standard library. It uses a local `.db` file (or `:memory:`) and exposes a connection and cursor API.

### 9.1 `sqlite3.connect()` — Connect to a database

**Description:** Open a SQLite database; create the file if it does not exist.

**Syntax**
```python
connection = sqlite3.connect("<database_file>.db")
```

**Example**
```python
import sqlite3

con = sqlite3.connect("INSTRUCTOR.db")
```

### 9.2 `con.cursor()` — Create a cursor

**Description:** Create an object used to run SQL statements and retrieve query results.

**Syntax**
```python
cursor_obj = connection.cursor()
```

**Example**
```python
cursor_obj = con.cursor()
```

### 9.3 `cursor.execute()` — Run one SQL statement

**Description:** Execute SQL. Use **parameter placeholders** for data values instead of interpolating untrusted strings into SQL.

**Syntax**
```python
cursor_obj.execute("<SQL statement>", (<parameter_1>, <parameter_2>))
```

**Example**
```python
cursor_obj.execute(
    "INSERT INTO instructor (id, first_name, last_name, city, country) "
    "VALUES (?, ?, ?, ?, ?)",
    (1, "Rav", "Ahuja", "Toronto", "CA")
)
con.commit()
```

**Note:** The `execute()` return value is a cursor, not a list of rows; use fetching methods for `SELECT` results.

### 9.4 `fetchall()` — Retrieve all remaining rows

**Description:** Fetch all rows left in the cursor's result as a list of tuples.

**Syntax**
```python
rows = cursor_obj.fetchall()
```

**Example**
```python
cursor_obj.execute("SELECT * FROM instructor")
output_all = cursor_obj.fetchall()

for row in output_all:
    print(row)
```

### 9.5 `fetchmany()` — Retrieve a specified batch

**Description:** Fetch up to the specified number of rows from the current cursor position.

**Syntax**
```python
rows = cursor_obj.fetchmany(<number_of_rows>)
```

**Example**
```python
cursor_obj.execute("SELECT * FROM instructor")
output_many = cursor_obj.fetchmany(2)

for row in output_many:
    print(row)
```

### 9.6 `con.commit()` — Save pending changes

**Description:** Commit inserts, updates, or deletes under normal SQLite transaction settings.

**Syntax**
```python
connection.commit()
```

**Example**
```python
con.commit()
```

### 9.7 `con.close()` — Release a SQLite connection

**Description:** Close the connection when finished. Closing is **not** a substitute for committing pending changes.

**Syntax**
```python
connection.close()
```

**Example**
```python
con.close()
```

### 9.8 End-to-end working SQLite example

```python
import sqlite3

con = sqlite3.connect("INSTRUCTOR.db")

try:
    cursor_obj = con.cursor()

    cursor_obj.execute("""
        CREATE TABLE IF NOT EXISTS instructor (
            id INTEGER PRIMARY KEY,
            first_name TEXT NOT NULL,
            last_name TEXT NOT NULL,
            city TEXT,
            country TEXT
        )
    """)

    cursor_obj.execute("""
        INSERT OR REPLACE INTO instructor
            (id, first_name, last_name, city, country)
        VALUES (?, ?, ?, ?, ?)
    """, (1, "Rav", "Ahuja", "Toronto", "CA"))

    con.commit()

    cursor_obj.execute("SELECT * FROM instructor")
    for row in cursor_obj.fetchall():
        print(row)
finally:
    con.close()
```

**Note:** `INSERT OR REPLACE` is a SQLite-specific conflict-handling form, used here so repeated runs do not fail on the example primary key. It replaces a conflicting row and can trigger delete-related effects; ordinary `INSERT` is preferable when replacement is not intended.

---

## 10. Pandas, CSV files, and visualizations

> These are **Python data-analysis functions**, not SQL language statements. They work alongside a database connection.

### 10.1 `pd.read_sql_query()` — SQL result to a DataFrame

**Description:** Execute a `SELECT` query and return a pandas DataFrame.

**Syntax**
```python
df = pd.read_sql_query("<SELECT query>", connection)
```

**Example**
```python
import pandas as pd

df = pd.read_sql_query("SELECT * FROM instructor", con)
```

### 10.2 `DataFrame.shape` — Dimensions of a DataFrame

**Description:** Return `(row_count, column_count)` as a tuple. This is an attribute, **not a function**.

**Syntax**
```python
(row_count, column_count) = dataframe.shape
```

**Example**
```python
print(df.shape)
```

### 10.3 `pd.read_csv()` — CSV to DataFrame

**Description:** Read a local or remote CSV file into pandas.

**Syntax**
```python
df = pd.read_csv("<path_or_url_to_csv>")
```

**Example**
```python
import pandas as pd

df = pd.read_csv("https://data.cityofchicago.org/resource/jcxq-k9xf.csv")
```

**Note:** The Chicago URL is an example remote dataset endpoint; it requires internet access, and the returned format/availability is external to pandas. For a local CSV, use `pd.read_csv("data.csv")`.

### 10.4 `DataFrame.to_sql()` — DataFrame to a SQL table

**Description:** Write a DataFrame into a database table using a supported connection. `if_exists='replace'` **drops and recreates the target table**, so use it carefully.

**Syntax**
```python
df.to_sql(
    name="<table_name>",
    con=connection,
    if_exists="fail",   # "fail", "replace", or "append"
    index=False
)
```

**Example**
```python
import pandas as pd

# Assuming df is an existing DataFrame and con is a sqlite3 connection:
df.to_sql("chicago_socioeconomic_data", con, if_exists="replace", index=False)
```

**Note:** pandas also supports SQLAlchemy database connections for other DBMSs.

### 10.5 `pd.read_sql()` — Read SQL query/table

**Description:** Load an SQL query result (or, depending on connection type, a table) into a DataFrame.

**Syntax**
```python
df = pd.read_sql("<SELECT query>", connection)
```

**Example**
```python
import pandas as pd

select_query = "SELECT * FROM instructor"
df = pd.read_sql(select_query, con)
```

### 10.6 `seaborn.barplot()` — Plot category vs. numeric value

**Description:** Create a bar chart; by default it displays an **estimated mean** numeric value per category, not automatically a frequency count.

**Syntax**
```python
sns.barplot(x="<category_column>", y="<numeric_column>", data=dataframe)
```

**Example — already aggregated frequency data**
```python
import seaborn as sns
import matplotlib.pyplot as plt

# dataframe must contain columns named "test_score" and "frequency".
sns.barplot(x="test_score", y="frequency", data=dataframe)
plt.xlabel("Test score")
plt.ylabel("Frequency")
plt.show()
```

**Tip:** For frequencies of **raw records**, use `sns.countplot(x="test_score", data=dataframe)` instead.

---

## 11. IBM Db2 with Python (`ibm_db`)

> `ibm_db` is a Db2-specific Python driver. It is **not** the same API as `sqlite3`; for example, close a connection with `ibm_db.close(conn)`.

### 11.1 `ibm_db.connect()` — Connect to Db2

**Description:** Establish a connection using a Db2 connection string. **Never commit actual credentials to GitHub.**

**Syntax**
```python
conn = ibm_db.connect(
    "DATABASE=<database>;HOSTNAME=<host>;PORT=<port>;"
    "PROTOCOL=TCPIP;UID=<username>;PWD=<password>;",
    "",
    ""
)
```

**Example — environment variables**
```python
import os
import ibm_db

conn_str = (
    f"DATABASE={os.environ['DB2_DATABASE']};"
    f"HOSTNAME={os.environ['DB2_HOST']};"
    f"PORT={os.environ['DB2_PORT']};"
    "PROTOCOL=TCPIP;"
    f"UID={os.environ['DB2_USER']};"
    f"PWD={os.environ['DB2_PASSWORD']};"
)
conn = ibm_db.connect(conn_str, "", "")
```

### 11.2 `ibm_db.server_info()` — Retrieve server metadata

**Description:** Obtain Db2 server information from a connection.

**Syntax**
```python
server = ibm_db.server_info(conn)
```

**Example**
```python
server = ibm_db.server_info(conn)
print("DBMS_NAME:", server.DBMS_NAME)
print("DBMS_VER:", server.DBMS_VER)
print("DB_NAME:", server.DB_NAME)
```

### 11.3 `ibm_db.exec_immediate()` — Execute SQL immediately

**Description:** Execute a statement without explicitly preparing it first. Use parameterized preparation for variable or untrusted input instead.

**Syntax**
```python
statement = ibm_db.exec_immediate(conn, "<SQL statement>")
```

**Example**
```python
statement = ibm_db.exec_immediate(
    conn,
    "SELECT emp_id, f_name FROM employees"
)
```

### 11.4 `ibm_db.fetch_tuple()` — Fetch query rows

**Description:** Retrieve one row as a tuple each time it is called; returns no row when exhausted.

**Syntax**
```python
row = ibm_db.fetch_tuple(statement)
```

**Example**
```python
row = ibm_db.fetch_tuple(statement)
while row:
    print(row)
    row = ibm_db.fetch_tuple(statement)
```

### 11.5 `ibm_db.close()` — Close a Db2 connection

**Description:** Release the connection opened by `ibm_db.connect()`.

**Syntax**
```python
ibm_db.close(conn)
```

**Example**
```python
ibm_db.close(conn)
```

**Note:** `conn.close()` applies to Python connection objects from drivers exposing that interface (such as `sqlite3` or `ibm_db_dbi`), **not** the low-level `ibm_db` connection returned by `ibm_db.connect()`.

---

## 12. Dialect differences and common mistakes

### Quick compatibility matrix

| Feature | MySQL | IBM Db2 (LUW) | SQLite |
| --- | --- | --- | --- |
| Limit rows | `LIMIT 5` | `FETCH FIRST 5 ROWS ONLY` | `LIMIT 5` |
| Change column type | `MODIFY COLUMN` | `ALTER COLUMN ... SET DATA TYPE` | Often rebuild table |
| Rename column | `RENAME COLUMN ... TO ...` | `RENAME COLUMN ... TO ...` | `RENAME COLUMN ... TO ...` (3.25+) |
| `TRUNCATE TABLE` | Yes | Yes (`IMMEDIATE` supported) | No; use `DELETE` |
| Full outer join | **No** (emulate) | Yes | Yes (3.39+) |
| Date difference in days | `DATEDIFF(d1, d2)` | `DAYS(d1) - DAYS(d2)` | `JULIANDAY(d1) - JULIANDAY(d2)` (consider times) |
| Add 3 days | `DATE_ADD(d, INTERVAL 3 DAY)` | `d + 3 DAYS` | `DATE(d, '+3 days')` |
| Upper/lowercase | `UPPER`, `LOWER` | `UPPER`, `LOWER` | `UPPER`, `LOWER` |
| Python connection close | Driver-dependent | `ibm_db.close(conn)` for `ibm_db` | `con.close()` |

### Common mistakes to avoid

1. **`COUNT *` is invalid:** use `COUNT(*)` for all rows or `COUNT(column)` for non-NULL values.
2. **`DISTINCT` is not a function:** write `SELECT DISTINCT column_name ...`.
3. **`UPDATE` / `DELETE` without `WHERE`:** can affect the entire table.
4. **Double quotes for text:** use standard SQL single quotes for strings: `'Rome'`.
5. **`WHERE column = NULL`:** use `IS NULL`.
6. **Unqualified columns in joins:** use aliases (`e.dep_id`, `d.dep_name`) to avoid ambiguity.
7. **Filtering right-side columns after a `LEFT JOIN`:** a `WHERE` condition can eliminate the unmatched rows you wanted to retain.
8. **No `ORDER BY` with `LIMIT`:** the returned subset has no guaranteed ordering.
9. **Using `FROM_DAYS(DATEDIFF(...))` as an age calculation:** it converts day counts to calendar dates, not age.
10. **Calling `ibm_db` and `sqlite3` methods interchangeably:** they have different connection, statement, and fetch APIs.
11. **Assuming all SQL dialects share date functions:** verify the engine before reusing date arithmetic.
12. **Hard-coding database credentials in GitHub:** use environment variables or a secret manager.
13. **Assuming `TRUNCATE` can always be rolled back:** this is not true in MySQL or Db2 `IMMEDIATE`.
14. **Assuming `UNION` and `UNION ALL` are equivalent:** `UNION` removes duplicate rows; `UNION ALL` does not.

### SQL query execution order (mental model)

Although written starting with `SELECT`, a useful conceptual order is:

```text
FROM / JOIN
    -> WHERE
    -> GROUP BY
    -> HAVING
    -> SELECT
    -> DISTINCT
    -> ORDER BY
    -> LIMIT / FETCH FIRST
```

This is a logical learning model, not a promise about the optimizer's physical execution plan.

---

## 13. Official references

- [MySQL 8.4 Reference — Date and time functions](https://dev.mysql.com/doc/refman/8.4/en/date-and-time-functions.html)
- [MySQL 8.4 Reference — ALTER TABLE](https://dev.mysql.com/doc/refman/8.4/en/alter-table.html)
- [MySQL 8.4 Reference — JOIN](https://dev.mysql.com/doc/refman/8.4/en/join.html)
- [IBM Db2 11.5 — ALTER TABLE](https://www.ibm.com/docs/en/db2/11.5.x?topic=statements-alter-table)
- [IBM Db2 11.5 — TRUNCATE](https://www.ibm.com/docs/en/db2/11.5.x?topic=statements-truncate)
- [IBM Db2 11.5 — Date arithmetic](https://www.ibm.com/docs/en/db2/11.5.x?topic=expressions-datetime-operations-durations)
- [SQLite — SELECT and joins](https://www.sqlite.org/lang_select.html)
- [SQLite — Date and time functions](https://www.sqlite.org/lang_datefunc.html)
- [Python documentation — `sqlite3`](https://docs.python.org/3/library/sqlite3.html)
- [pandas — `read_sql_query`](https://pandas.pydata.org/docs/reference/api/pandas.read_sql_query.html)
- [pandas — `DataFrame.to_sql`](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_sql.html)
- [IBM — Python `ibm_db` connection](https://www.ibm.com/docs/en/db2/11.5.x?topic=db-connecting-database-server)

---

**Suggested GitHub filename:** `SQL_Cheatsheet.md` (or `README.md` if this is the main content of the repository).

**Scope:** Reference notes and illustrative queries; test DDL and destructive DML in a disposable database before using them on real data.
