# ADVANCED WINDOW FUNCTIONS

## Quick Definition

A **window function** performs a calculation across a set of related rows while **keeping every individual row** in the result.

That is the key difference from `GROUP BY`:

```text
GROUP BY
Many rows → fewer rows

Window function
Many rows → same number of rows
```

Window functions are extremely useful for rankings, running totals, comparisons with previous/next rows, percentages, and advanced reporting.

---

## The Basic Idea

Suppose we have:

| id | employee | department | salary |
| -: | -------- | ---------- | -----: |
|  1 | Ali      | IT         |  60000 |
|  2 | Sara     | IT         |  80000 |
|  3 | John     | HR         |  50000 |
|  4 | Ahmed    | HR         |  70000 |

With `GROUP BY`:

```sql
SELECT
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

You get:

| department | avg_salary |
| ---------- | ---------: |
| IT         |      70000 |
| HR         |      60000 |

The individual employees disappear.

With a window function:

```sql
SELECT
    employee,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg
FROM employees;
```

Result:

| employee | department | salary | department_avg |
| -------- | ---------- | -----: | -------------: |
| Ali      | IT         |  60000 |          70000 |
| Sara     | IT         |  80000 |          70000 |
| John     | HR         |  50000 |          60000 |
| Ahmed    | HR         |  70000 |          60000 |

Every employee remains.

---

# Syntax

The basic structure is:

```sql
FUNCTION() OVER (
    PARTITION BY column
    ORDER BY column
)
```

Example:

```sql
SELECT
    employee,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

Important parts:

`OVER()` → tells SQL this is a window calculation.

`PARTITION BY` → divides rows into groups.

`ORDER BY` → determines the order inside the window.

`ROWS` / `RANGE` → controls which rows are included in a window frame.

---

# 1. PARTITION BY

`PARTITION BY` creates separate windows.

```sql
SELECT
    employee,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg
FROM employees;
```

Conceptually:

```text
IT
 ├── Ali     60000
 └── Sara    80000

HR
 ├── John    50000
 └── Ahmed   70000
```

The average is calculated separately for IT and HR.

Without `PARTITION BY`:

```sql
AVG(salary) OVER ()
```

the average is calculated across **all employees**.

---

# 2. Ranking Functions

The most important ranking functions are:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
```

They look similar but behave differently.

## ROW_NUMBER()

Gives every row a unique number.

```sql
SELECT
    employee,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS row_num
FROM employees;
```

Result:

| employee | salary | row_num |
| -------- | -----: | ------: |
| Sara     |  80000 |       1 |
| Ahmed    |  70000 |       2 |
| Ali      |  60000 |       3 |
| John     |  50000 |       4 |

Even if salaries are equal, each row gets a different number.

---

## RANK()

Equal values receive the same rank, and gaps appear afterward.

Suppose:

| employee | salary |
| -------- | -----: |
| Sara     |  80000 |
| Ahmed    |  70000 |
| Ali      |  70000 |
| John     |  50000 |

```sql
SELECT
    employee,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

Result:

| employee | salary | salary_rank |
| -------- | -----: | ----------: |
| Sara     |  80000 |           1 |
| Ahmed    |  70000 |           2 |
| Ali      |  70000 |           2 |
| John     |  50000 |           4 |

Notice rank `3` is skipped.

---

## DENSE_RANK()

Same ranking for ties, but **no gaps**.

```sql
SELECT
    employee,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

Result:

| employee | salary | salary_rank |
| -------- | -----: | ----------: |
| Sara     |  80000 |           1 |
| Ahmed    |  70000 |           2 |
| Ali      |  70000 |           2 |
| John     |  50000 |           3 |

Remember:

```text
ROW_NUMBER()
1, 2, 3, 4

RANK()
1, 2, 2, 4

DENSE_RANK()
1, 2, 2, 3
```

---

# 3. Ranking Within Each Group

This is where window functions become powerful.

Find each employee's salary rank **inside their department**:

```sql
SELECT
    employee,
    department,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS department_rank
FROM employees;
```

Example:

| employee | department | salary | department_rank |
| -------- | ---------- | -----: | --------------: |
| Sara     | IT         |  80000 |               1 |
| Ali      | IT         |  60000 |               2 |
| Ahmed    | HR         |  70000 |               1 |
| John     | HR         |  50000 |               2 |

The window resets for every department.

---

# 4. LAG()

`LAG()` lets you access a value from a **previous row**.

Suppose we have monthly sales:

| month | sales |
| ----- | ----: |
| Jan   |  1000 |
| Feb   |  1200 |
| Mar   |  1500 |
| Apr   |  1300 |

```sql
SELECT
    month,
    sales,
    LAG(sales) OVER (
        ORDER BY month
    ) AS previous_sales
FROM monthly_sales;
```

Result:

| month | sales | previous_sales |
| ----- | ----: | -------------: |
| Jan   |  1000 |           NULL |
| Feb   |  1200 |           1000 |
| Mar   |  1500 |           1200 |
| Apr   |  1300 |           1500 |

Now you can calculate the change:

```sql
SELECT
    month,
    sales,
    sales - LAG(sales) OVER (
        ORDER BY month
    ) AS sales_change
FROM monthly_sales;
```

Result:

| month | sales | sales_change |
| ----- | ----: | -----------: |
| Jan   |  1000 |         NULL |
| Feb   |  1200 |          200 |
| Mar   |  1500 |          300 |
| Apr   |  1300 |         -200 |

---

# 5. LEAD()

`LEAD()` does the opposite.

It looks at the **next row**.

```sql
SELECT
    month,
    sales,
    LEAD(sales) OVER (
        ORDER BY month
    ) AS next_sales
FROM monthly_sales;
```

Result:

| month | sales | next_sales |
| ----- | ----: | ---------: |
| Jan   |  1000 |       1200 |
| Feb   |  1200 |       1500 |
| Mar   |  1500 |       1300 |
| Apr   |  1300 |       NULL |

Easy way to remember:

```text
LAG  → look backward
LEAD → look forward
```

---

# 6. Running Total

Window functions can calculate a cumulative total.

```sql
SELECT
    month,
    sales,
    SUM(sales) OVER (
        ORDER BY month
    ) AS running_total
FROM monthly_sales;
```

Result:

| month | sales | running_total |
| ----- | ----: | ------------: |
| Jan   |  1000 |          1000 |
| Feb   |  1200 |          2200 |
| Mar   |  1500 |          3700 |
| Apr   |  1300 |          5000 |

The calculation progresses row by row:

```text
1000
1000 + 1200 = 2200
2200 + 1500 = 3700
3700 + 1300 = 5000
```

---

# 7. Window Frames

For advanced window functions, you need to understand the **window frame**.

Consider:

```sql
SUM(sales) OVER (
    ORDER BY month
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

This means:

> Start from the first row and continue through the current row.

That's a running total.

You can also create a moving average:

```sql
SELECT
    month,
    sales,
    AVG(sales) OVER (
        ORDER BY month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg
FROM monthly_sales;
```

For each row, SQL considers:

```text
current row
+ previous row
+ row before previous
```

So it creates a 3-row moving average.

---

# 8. FIRST_VALUE() and LAST_VALUE()

`FIRST_VALUE()` returns the first value in the window.

```sql
SELECT
    employee,
    salary,
    FIRST_VALUE(salary) OVER (
        ORDER BY salary DESC
    ) AS highest_salary
FROM employees;
```

Every row can now see the highest salary.

`LAST_VALUE()` requires more care because the default window frame can produce surprising results.

A safer explicit frame is:

```sql
SELECT
    employee,
    salary,
    LAST_VALUE(salary) OVER (
        ORDER BY salary
        ROWS BETWEEN UNBOUNDED PRECEDING
        AND UNBOUNDED FOLLOWING
    ) AS lowest_salary
FROM employees;
```

For advanced SQL, understanding the window frame matters here.

---

# 9. Percentage of Total

Window functions are excellent for calculating percentages.

Suppose:

| product | sales |
| ------- | ----: |
| Laptop  |  5000 |
| Phone   |  3000 |
| Tablet  |  2000 |

Calculate each product's percentage of total sales:

```sql
SELECT
    product,
    sales,
    sales * 100.0 / SUM(sales) OVER () AS percentage_of_total
FROM product_sales;
```

Result:

| product | sales | percentage_of_total |
| ------- | ----: | ------------------: |
| Laptop  |  5000 |                  50 |
| Phone   |  3000 |                  30 |
| Tablet  |  2000 |                  20 |

Notice:

```sql
SUM(sales) OVER ()
```

calculates the total while keeping every product row.

---

# 10. Top N Per Group

This is one of the most important real-world window-function patterns.

Suppose you want the **top 2 highest-paid employees in every department**.

First rank them:

```sql
WITH ranked_employees AS (
    SELECT
        employee,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT
    employee,
    department,
    salary
FROM ranked_employees
WHERE rn <= 2;
```

The important part is:

```sql
ROW_NUMBER() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

Then the outer query filters the ranking.

You can't normally use the window-function alias directly in `WHERE` at the same query level, which is why the CTE is useful.

---

# Common Mistakes

## Mistake #1 — Confusing GROUP BY with window functions

`GROUP BY`:

```sql
SELECT
    department,
    AVG(salary)
FROM employees
GROUP BY department;
```

Produces one row per department.

Window function:

```sql
SELECT
    employee,
    department,
    AVG(salary) OVER (
        PARTITION BY department
    )
FROM employees;
```

Keeps every employee.

---

## Mistake #2 — Forgetting ORDER BY for ranking

This:

```sql
ROW_NUMBER() OVER ()
```

doesn't define a meaningful order.

For ranking, specify one:

```sql
ROW_NUMBER() OVER (
    ORDER BY salary DESC
)
```

---

## Mistake #3 — Using ROW_NUMBER when ties should share a rank

If two employees have the same salary:

```sql
ROW_NUMBER()
```

still gives them different numbers.

Use:

```sql
RANK()
```

or:

```sql
DENSE_RANK()
```

if equal values should share a rank.

---

## Mistake #4 — Trying to use a window function directly in WHERE

This generally doesn't work:

```sql
SELECT
    employee,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS rn
FROM employees
WHERE rn <= 3;
```

Use a CTE:

```sql
WITH ranked AS (
    SELECT
        employee,
        salary,
        ROW_NUMBER() OVER (
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

---

# Performance Notes

Window functions can require the database to sort and process many rows.

This:

```sql
RANK() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

may require sorting employees within departments.

For large datasets:

* Filter unnecessary rows before the window calculation when possible.
* Indexes can sometimes help with filtering and ordering, depending on the database and query.
* Don't calculate several expensive windows if you only need one.
* Use `EXPLAIN` / execution plans when performance matters.
* Be especially careful with large partitions and large window frames.

Window functions are powerful, but they aren't magic performance shortcuts.

---

# Practice

### Exercise 1 — Easy

Rank all employees by salary from highest to lowest.

```sql id="b5khk8"
-- Your query here
```

**Answer:**

```sql id="1m9czf"
SELECT
    employee,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

### Exercise 2 — Medium

Calculate each employee's average department salary.

```sql id="t8l4yd"
-- Your query here
```

**Answer:**

```sql id="n7n0pu"
SELECT
    employee,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_avg
FROM employees;
```

### Exercise 3 — Hard

Find the top 3 employees by salary in each department.

**Answer:**

```sql id="l1rjzu"
WITH ranked AS (
    SELECT
        employee,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT
    employee,
    department,
    salary
FROM ranked
WHERE rn <= 3;
```

### Exercise 4 — Advanced

Calculate monthly sales and the change from the previous month.

```sql id="gk4i4d"
SELECT
    month,
    sales,
    sales - LAG(sales) OVER (
        ORDER BY month
    ) AS sales_change
FROM monthly_sales;
```

---

# FAQ

**Q: What makes a window function different from GROUP BY?**

`GROUP BY` reduces rows.

Window functions keep the original rows.

```text
GROUP BY:
10 rows → 3 rows

Window function:
10 rows → 10 rows
```

**Q: What does PARTITION BY do?**

It divides the data into independent groups for the window calculation.

```sql id="0o5a2y"
AVG(salary) OVER (
    PARTITION BY department
)
```

Each department gets its own average.

**Q: What's the difference between RANK and DENSE_RANK?**

With salaries:

```text
80000
70000
70000
60000
```

`RANK()`:

```text
1
2
2
4
```

`DENSE_RANK()`:

```text
1
2
2
3
```

**Q: What's the difference between LAG and LEAD?**

```text
LAG  → previous row
LEAD → next row
```

They are commonly used for period-over-period comparisons.

**Q: Can aggregate functions be used as window functions?**

Yes.

For example:

```sql id="e0khbe"
SUM(amount) OVER (...)
AVG(amount) OVER (...)
COUNT(*) OVER (...)
```

This is one of the most useful features of window functions.

**Q: Why do I need a CTE with ROW_NUMBER?**

Because you often need to filter based on the window result.

For example:

```sql id="5xwq9f"
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            ORDER BY salary DESC
        ) AS rn
    FROM employees
)
SELECT *
FROM ranked
WHERE rn <= 5;
```

**Q: What is a window frame?**

It defines exactly which rows are included in the calculation.

For example:

```sql id="xq8k3z"
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

means:

> Current row + previous two rows.

This is commonly used for moving averages and rolling calculations.

---

# Key Takeaways

* 🔹 Window functions calculate across related rows **without collapsing them**.
* 🔹 `PARTITION BY` creates separate calculation groups.
* 🔹 `ORDER BY` controls the order inside the window.
* 🔹 `ROW_NUMBER`, `RANK`, and `DENSE_RANK` handle ranking.
* 🔹 `LAG` and `LEAD` compare previous/next rows.
* 🔹 `SUM() OVER()` is excellent for running totals and percentages.
* 🔹 Window frames control exactly which rows participate in a calculation.

---

# Next Steps

You now have the core advanced window-function toolkit:

`ROW_NUMBER()` → unique ranking
`RANK()` / `DENSE_RANK()` → ranking with ties
`LAG()` / `LEAD()` → previous/next row comparisons
`SUM() OVER()` → running totals
`AVG() OVER()` → moving averages
`PARTITION BY` → calculations per group
`Window frames` → precise rolling calculations
`CTE + Window Function` → top-N-per-group and advanced reporting
