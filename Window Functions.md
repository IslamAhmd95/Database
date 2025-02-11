-- -

### What Are Window Functions?

A **window function** in SQL performs calculations **across a set of rows** related to the current row **without collapsing the result into a single value**. This means each row **retains its original details**, but an extra calculated value is added.

#### Types of Window Functions

SQL provides different types of window functions:

1. **Ranking Functions** (Assign ranks to rows)
    - `ROW_NUMBER()`: Assigns a unique row number to each record.
    - `RANK()`: Assigns ranks, but allows ties (same rank for same values).
    - `DENSE_RANK()`: Similar to `RANK()`, but without skipping ranks for ties.
2. **Aggregate Window Functions** (Perform calculations over a subset of rows)
    - `SUM()`: Running total over a window.
    - `AVG()`: Running average over a window.
    - `COUNT()`: Running count of rows.
3. **Value Functions** (Get values from other rows)
    - `LAG()`: Get the value from the previous row.
    - `LEAD()`: Get the value from the next row.
    - `FIRST_VALUE()`: Get the first value in the window.
    - `LAST_VALUE()`: Get the last value in the window.

---
### Ranking Functions 

Ranking functions **assign a ranking or row number** to each row based on some ordering criteria.

1. **ROW_NUMBER():** Assigns a unique row number to each row in the result set.
2. **RANK():** Assigns a rank to each row, but **allows ties**. **If two rows have the same value, they get the same rank, and the next rank is skipped.**
3. **DENSE_RANK():** Similar to `RANK()`, but **doesn't skip numbers when there is a tie**.
4. **NTILE():** Divides rows into **n equal groups** (or as close to equal as possible).


**PARTITION BY:** used to divide the result set into partitions and perform computation on each subset of partitioned data.


**Examples:**
```sql

-- ROW_NUMBER()
SELECT student_name, score, 
ROW_NUMBER() OVER(ORDER BY score DESC) AS row_num
FROM students;
-- The output will be the students ordered descending with a new field "row_num" which has the order of the students from the highest score to the lowest 
-- row_num -> "1 2 3 4 5 ..."

-- **Key Points:**
	-- **Always unique** (even if scores are the same).
	-- **Sequential numbers (no skipping).**
	-- **If you rerun the query, the order might change for ties.**


-- RANK()
SELECT student_name, score, 
RANK() OVER(ORDER BY score DESC) AS rank_num
FROM students;
-- The output will be the students ordered descending with a new field "rank_num" which has the rank of the students from the highest score to the lowest 
-- rank_num -> "1 2 3 3 3 6 ..."

-- **Key Points:**
	-- **Alice, Charlie, and Eve get rank 3 (same score).**
	-- **The next rank skips to 6 (3+3=6) instead of 4 because of the tie.**


-- DENSE_RANK()
SELECT student_name, score,
DENSE_RANK() OVER(ORDER BY score DESC) AS dense_rank_num
FROM students;
-- The output will be the students ordered descending with a new field "rank_num" which has the rank of the students from the highest score to the lowest 
-- dense_rank_num -> "1 2 3 3 3 4 ..."

-- **Key Points:**
	-- **Alice, Charlie, and Eve share rank 3 (same score).**
	-- **No gap in ranking (next rank is 4, not 6).**


-- NTILE(n)
SELECT student_name, score,
NTILE(3) OVER(ORDER BY score DESC) AS tile_num
FROM students;
-- The output will be 3 groups of students ordered descending with a new field "tile_num" which has the the same number for each student of the same group of students 
-- tile_num -> 1 for group 1 of scores (95, 92), 2 for group 2 of scores (87, 85, 81), 3 for group 3 of scores (77, 71) ... 

-- **Key Points:**
	-- **Divides rows into 3 equal parts (or as close as possible).**
	-- **Each row is assigned a group number (1, 2, or 3).**


-- NTILE() with PARTITION BY
-- Suppose we have employees in different **departments**, and we want to divide **each department’s employees** into **4 salary groups** separately.
SELECT employee_name, department, salary,
NTILE(4) OVER(PARTITION BY department ORDER BY salary DESC) AS bucket
FROM employees;
-- **Key Takeaways:**
	-- The number inside `NTILE(n)` represents the number of groups (buckets), NOT the number of items per group.
	-- Employees are divided **into 4 groups per department**.
	-- **Each department is handled separately.**
	-- Without `PARTITION BY`, **all employees** would be ranked together.
	-- **Even if there are only 2 departments, each department is processed separately!**  
	-- **`NTILE(4)` still creates 4 groups in each department**, even if there aren't many employees.  
	-- If a department has fewer than 4 employees, it **still tries to divide them** but may result in some empty groups.
	-- **If the number of rows is not evenly divisible by `n`, some groups will have one extra row.**


-- RANK() with PARTITION BY
-- **Find the rank of employees within each department based on salary.**
SELECT employee_name, department, salary,
RANK() OVER(PARTITION BY department ORDER BY salary desc) AS dept_rank
FROM employees;
-- **Key Takeaways:**
	-- **Ranks are assigned within each department** separately.
	-- Without `PARTITION BY`, the ranking would be across **all departments**.

-- ROW_NUMBER() with PARTITION BY
-- **Find the top 2 highest-paid employees in each department.**
SELECT * FROM
(
	SELECT employee_name, department, salary,
	ROW_NUMBER() OVER(PARTITION BY department ORDER BY salary DESC) AS row_num
	FROM employees
) ranked_employees where row_num <= 2;
-- **Key Takeaways:**
	-- We first assign a `ROW_NUMBER()` inside each department.
	-- Then, we filter (`WHERE row_num <= 2`) to keep only **the top 2 employees per department**.
```

---

### Value Functions

Value window functions **return specific row values** instead of calculating ranks or aggregates. These functions help in accessing previous, next, or other reference values within a partition.

1. LEAD(column): **Gets the value from the next row** in a window.
2. LAG(column): **Gets the value from the previous row** in a window.
3. FIRST_VALUE(column): **Returns the first value in the partition**.
4. NTH_VALUE(column, N): **Retrieves the Nth value** from the ordered window.
5. LAST_VALUE(column): **Returns the last value in the partition**. Unlike `FIRST_VALUE()`, `LAST_VALUE()` is affected by the current window frame.

#### Window Frame
**Without this frame**, window functions **only consider rows up to the current row** by default.
It **can be used with many window functions**, but it is **most commonly seen with `LAST_VALUE()`**.

##### Common Options
- **UNBOUNDED PRECEDING:** Start from the first row in the partition
- **UNBOUNDED FOLLOWING:** Extends to the last row in the partition
- **CURRENT ROW:** Includes the current row
- **X PRECEDING:** X rows before the current row
- **X FOLLOWING:** X rows AFTER the current row


**Examples:**
```sql
-- LEAD()
-- Comparing Employees' Salaries with the Next Employee
SELECT name, department, salary,
LEAD(salary) OVER(PARTITION BY department ORDER BY salary DESC) AS next_salary
FROM employees;
-- **Key Takeaways:**
	-- The function moves **down** one row and retrieves the **next salary**.
	-- **`NULL` appears for the last row** because there's no next row.


-- LAG()
-- Checking the Previous Employee’s Salary
SELECT name, department, salary,
LAG(salary) OVER(PARTITION BY department ORDER BY salary DESC) AS prev_salary
FROM employees;
-- **Key Takeaways:**
	-- The function moves **up** one row and retrieves the **previous salary**.
	-- **`NULL` appears for the first row** because there's no previous row.


-- FIRST_VALUE()
-- Retrieve the highest salary in each department
SELECT name, department, salary,
FIRST_VALUE(salary) OVER(PARTITION BY department ORDER BY salary DESC) AS highest_salary
FROM employees;
-- **Key Takeaways:**

	-- **First value stays the same** for all rows in the partition.
	-- It finds the **highest salary per department** based on `ORDER BY salary DESC`.


-- NTH_VALUE()
-- Getting the 3rd Highest Salary
SELECT name, department, salary,
NTH_VALUE(salary, 3) OVER(PARTITION BY department ORDER BY salary DESC) AS
third_highest_salary
FROM employees;
-- **Key Takeaways:**
	-- `NTH_VALUE(salary, 3)` → Takes the **3rd highest salary**.
	-- The result is **the same for all rows** in that department **because it's a window function**.
	-- If there are fewer than 3 values, it returns **NULL**.


-- LAST_VALUE()
-- Getting the Lowest Salary in Each Department
SELECT name, department, salary,
LAST_VALUE(salary) OVER(PARTITION BY department ODER BY salary DESC ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS lowest_value
FROM employees;
-- **Key Takeaways:**
	-- `LAST_VALUE()` normally **returns the last row in the window frame**.
	-- To make it work **like `FIRST_VALUE()`**, we **set the frame to cover all rows** using: "ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING"
	

-- Window Frame with AVG()
-- calculate the department average salary for all rows.
SELECT employee_id, salary,
       AVG(salary) OVER (PARTITION BY department ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS department_avg
FROM Employees;


-- Window Frame with SUM()
-- Creates a rolling sum of "current row + previous 2 rows".
SELECT employee_id, salary,
       SUM(salary) OVER (PARTITION BY department ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_sum
FROM Employees;
```

---

### Aggregate Window Functions

Unlike normal aggregate functions (like `SUM()`, `AVG()`, etc.), which collapse rows into a **single result**, **windowed aggregate functions retain all rows** while calculating over a defined window.


**Normal aggregate functions (`AVG()`, `SUM()`, etc.)** **require** `GROUP BY` because they collapse multiple rows into a single result per group.
**Window aggregate functions (`AVG() OVER()`, `SUM() OVER()`, etc.)** **do NOT require** `GROUP BY` because they keep all rows while calculating results over a window.

```sql
-- Normal Aggregation: Collapses rows into one result per group
SELECT department, AVG(salary) 
FROM employees 
GROUP BY department;

-- Window Aggregation: Calculates average per department but retains all rows
SELECT name, department, salary, 
       AVG(salary) OVER (PARTITION BY department) AS avg_salary
FROM employees;
```


#### List of Aggregate Window Functions
- **SUM():** Total value in the window.
- **AVG():** Average of values in the window.
- **COUNT:** Number of values in the window.
- **MIN():** Smallest value in the window.
- **MAX():** Largest value in the window.

**Examples&Notes:**
```sql
-- AVG()
-- **Keeps all rows** while calculating an average **per partition**.
SELECT name, department, salary, 
       AVG(salary) OVER (PARTITION BY department) AS avg_salary
FROM employees;


-- SUM()
-- **Keeps all rows** while calculating **department-wide salary totals**.
SELECT name, department, salary, 
       SUM(salary) OVER (PARTITION BY department) AS total_salary
FROM employees;

SELECT salesperson, sales, SUM(sales) OVER (ORDER BY salesperson ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS running_total FROM sales_data;

-- COUNT()
-- **Keeps all rows** while counting **employees per department**.
SELECT name, department, salary, 
       COUNT(*) OVER (PARTITION BY department) AS employee_count
FROM employees;


-- MIN(), MAX()
-- **Keeps all rows** while showing **min/max salary per department**.
SELECT name, department, salary, 
       MIN(salary) OVER (PARTITION BY department) AS min_salary, 
       MAX(salary) OVER (PARTITION BY department) AS max_salary
FROM employees;


-- Window Functions with WHERE clause
-- ❌ **No, you can't directly use a window function inside `WHERE`**  
-- ✅ **But you can use a subquery or a CTE as a workaround.**
SELECT name, salary, 
       AVG(salary) OVER () AS avg_salary
FROM employees
WHERE avg_salary > 5000;  -- ❌ Error! Can't use window function in WHERE

SELECT * FROM (
    SELECT name, salary, 
           AVG(salary) OVER () AS avg_salary
    FROM employees
) subquery
WHERE avg_salary > 5000;  -- ✅ Works!


-- Window Function with HAVING clause
-- ❌ **No, `HAVING` only works with `GROUP BY`.**  
-- ✅ **Use a subquery or CTE instead.**
SELECT department, AVG(salary) OVER (PARTITION BY department) AS avg_salary
FROM employees
HAVING avg_salary > 5000;  -- ❌ Error!

WITH temp AS (
    SELECT department, salary, 
           AVG(salary) OVER (PARTITION BY department) AS avg_salary
    FROM employees
)
SELECT * FROM temp
WHERE avg_salary > 5000;  -- ✅ Works!


-- Window Function with ORDER BY
-- ✅ You can `ORDER BY` a window function.
SELECT name, salary, 
       AVG(salary) OVER () AS avg_salary
FROM employees
ORDER BY avg_salary DESC;  -- ✅ Works!


-- Window Functions with FILTER "PostgreSQL"
SELECT name, department, salary,
SUM(salary) FILTER(WHERE department='IT')
OVER () AS total_it_salary FROM employees;
```

---

### WINDOW ... AS ...
The `WINDOW ... AS ...` clause is a **shortcut** that allows you to define **named window specifications** in SQL. Instead of repeating the same `PARTITION BY` and `ORDER BY` logic multiple times, you can **define the window once** using `WINDOW` and reuse it in multiple window functions.

#### Why Use `WINDOW ... AS ...`?
✅ **Avoid Repetition** – Define the window once and use it multiple times.  
✅ **Better Readability** – Makes complex queries cleaner.  
✅ **Easier Maintenance** – If you need to modify the window logic, change it in one place.

**Syntax&Examples:**
```sql
-- Syntax
SELECT column1, column2, ..,
window_function() OVER window_name
FROM table_name
WINDOW window_name AS (PARTITION BY columnX ORDER BY columnY);


-- total and average sales
SELECT salesperson, region, sales,
SUM(sales) OVER my_window AS running_total,
AVG(sales) OVER my_window AS avg_sales
FROM sales_data
WINDOW my_window AS (PARTITION BY region ORDER BY sales);


-- Example
-- Salespeople are **ranked within their region**.
-- Sales are **divided into 3 groups** within each region.
SELECT salesperson,
       region,
       sales,
       RANK() OVER my_window AS rank_in_region,
       NTILE(3) OVER my_window AS group_number
FROM sales_data
WINDOW my_window AS (PARTITION BY region ORDER BY sales DESC);


-- **Window functions execute after `HAVING`**, so you still **need a subquery** if you want to filter based on a window function.
-- **`HAVING` is for filtering GROUPS after `GROUP BY`**.
SELECT region,
       salesperson,
       sales,
       SUM(sales) OVER my_window AS total_sales
FROM sales_data
WINDOW my_window AS (PARTITION BY region)
HAVING total_sales > 10000;  -- ❌ ERROR: column "total_sales" does not exist having working only with GROUP BY clause

-- **computed `total_sales` first** in the subquery, so we can **use `WHERE`** in the outer query.
WITH sales_ranked AS (
    SELECT region,
           salesperson,
           sales,
           SUM(sales) OVER my_window AS total_sales
    FROM sales_data
    WINDOW my_window AS (PARTITION BY region)
)
SELECT * FROM sales_ranked
WHERE total_sales > 10000;  -- ✅ Now it works!
```

---

### Exercises

#### Find Employees with Above-Average Salary
```sql
SELECT * FROM
(
	SELECT *, AVG(salary) OVER() AS average_salary FROM employees
) employees_average
WHERE salary > average_salary;
```

#### Find the Highest Salary per Department
```sql
SELECT *, 
FIRST_VALUE(salary) OVER(PARTITION BY department ORDER BY salary DESC) AS 
dept_highest_salary
FROM employees;
```

#### Running Total of Sales But Only for a Certain Year (2023)
```sql
-- mysql & postgresql
WITH year_total_sales AS (
    SELECT *, 
           SUM(sales) OVER (ORDER BY salesperson) AS total_sales
    FROM sales_table 
    WHERE year = 2023;  -- Filter inside the CTE
) 
SELECT * FROM year_total_sales;

SELECT *,
    SUM(sales) OVER (ORDER BY salesperson) AS total_sales
FROM sales_table
WHERE year = 2023;

-- mysql
-- This includes all years in the output not only 2023, but other years return 0
SELECT *,
    SUM(CASE WHEN year = 2023 THEN sales ELSE 0 END) 
    OVER(ORDER BY salesperson) AS total_sales
FROM sales_table;

-- postgresql
-- This includes all years in the output not only 2023, but other years return 0
SELECT *,
	SUM(sales) FILTER(WHERE year='2023')
	OVER(ORDER BY salesperson) AS total_sales
FROM sales_table;
```