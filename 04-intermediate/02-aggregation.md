# SQL Aggregation

## Quick Definition

**Aggregation** means taking multiple rows and calculating a **single summary value** from them.

Common aggregate functions are:

`COUNT()` → count rows
`SUM()` → add values
`AVG()` → calculate average
`MIN()` → smallest value
`MAX()` → largest value

Think of aggregation as:

> **Many rows → one summary**

For example, instead of looking at every order, you can ask: “How many orders are there?” or “What is the total revenue?”

---

### The Basic Idea

Suppose we have an `orders` table:

| id | customer | amount |
| -: | -------- | -----: |
|  1 | Ali      |    100 |
|  2 | John     |    250 |
|  3 | Ali      |    150 |
|  4 | Sara     |    300 |

You can calculate the total:

```sql
SELECT SUM(amount)
FROM orders;
```

Result:

| SUM(amount) |
| ----------: |
|         800 |

Four rows became **one result**.

---

### Syntax

```sql
SELECT AGGREGATE_FUNCTION(column_name)
FROM table_name;
```

Example:

```sql
SELECT COUNT(*)
FROM users;
```

Common functions:

```sql
SELECT COUNT(*) FROM users;
SELECT SUM(amount) FROM orders;
SELECT AVG(amount) FROM orders;
SELECT MIN(amount) FROM orders;
SELECT MAX(amount) FROM orders;
```

---

## Examples

### Beginner — COUNT

Count all users:

```sql
SELECT COUNT(*)
FROM users;
```

Example result:

| COUNT(*) |
| -------: |
|      100 |

`COUNT(*)` counts rows.

You can also give the result a meaningful name:

```sql
SELECT COUNT(*) AS total_users
FROM users;
```

Result:

| total_users |
| ----------: |
|         100 |

---

### Beginner — SUM, AVG, MIN, MAX

Suppose `orders` contains:

| id | amount |
| -: | -----: |
|  1 |    100 |
|  2 |    250 |
|  3 |    150 |
|  4 |    300 |

Total:

```sql
SELECT SUM(amount) AS total_sales
FROM orders;
```

Average:

```sql
SELECT AVG(amount) AS average_order
FROM orders;
```

Smallest order:

```sql
SELECT MIN(amount) AS smallest_order
FROM orders;
```

Largest order:

```sql
SELECT MAX(amount) AS largest_order
FROM orders;
```

You can also use several aggregations together:

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(amount) AS total_sales,
    AVG(amount) AS average_order,
    MIN(amount) AS smallest_order,
    MAX(amount) AS largest_order
FROM orders;
```

---

### Intermediate — Aggregation with WHERE

Aggregation can be combined with filtering.

Find the total sales from orders greater than 100:

```sql
SELECT SUM(amount) AS total_sales
FROM orders
WHERE amount > 100;
```

The process is roughly:

```text
orders
   ↓
WHERE amount > 100
   ↓
remaining rows
   ↓
SUM(amount)
   ↓
one result
```

Another example:

```sql
SELECT AVG(amount) AS average_order
FROM orders
WHERE customer = 'Ali';
```

This calculates Ali's average order amount.

---

### Advanced — Aggregation with GROUP BY

This is where aggregation becomes much more useful.

Suppose:

| id | customer | amount |
| -: | -------- | -----: |
|  1 | Ali      |    100 |
|  2 | John     |    250 |
|  3 | Ali      |    150 |
|  4 | Sara     |    300 |
|  5 | John     |    100 |

If you want total spending **for each customer**:

```sql
SELECT
    customer,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer;
```

Result:

| customer | total_spent |
| -------- | ----------: |
| Ali      |         250 |
| John     |         350 |
| Sara     |         300 |

Without `GROUP BY`:

```sql
SELECT SUM(amount)
FROM orders;
```

You get **one total**.

With `GROUP BY`:

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer;
```

You get **one total per customer**.

That's the key idea:

```text
Aggregation alone
Many rows → one result

Aggregation + GROUP BY
Many rows → one result per group
```

---

## Common Mistakes

### Mistake #1: Selecting a normal column without GROUP BY

This is problematic:

```sql
SELECT customer, SUM(amount)
FROM orders;
```

You're asking SQL for a single total while also asking it which customer that total belongs to.

Correct:

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer;
```

### Mistake #2: Using WHERE to filter an aggregate

Wrong:

```sql
SELECT customer, SUM(amount)
FROM orders
WHERE SUM(amount) > 500
GROUP BY customer;
```

`WHERE` works before aggregation.

For filtering aggregate results, use `HAVING`:

```sql
SELECT customer, SUM(amount) AS total_spent
FROM orders
GROUP BY customer
HAVING SUM(amount) > 500;
```

### Mistake #3: Confusing COUNT(*) and COUNT(column)

```sql
SELECT COUNT(*)
FROM users;
```

Counts rows.

```sql
SELECT COUNT(phone)
FROM users;
```

Counts non-`NULL` values in `phone`.

So if 100 users exist but 20 have `phone = NULL`:

```text
COUNT(*)       = 100
COUNT(phone)   = 80
```

---

## Performance Notes

Aggregation can become expensive on large tables because the database may need to process many rows.

Indexes can sometimes help, especially when you filter first:

```sql
SELECT SUM(amount)
FROM orders
WHERE customer_id = 10;
```

An index on `customer_id` may help the database find the relevant orders efficiently.

For large datasets, avoid calculating unnecessary aggregations over millions of rows when you only need a small subset.

---

## Practice

### Exercise 1 — Easy

Count all products.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT COUNT(*) AS total_products
FROM products;
```

### Exercise 2 — Medium

Find the total and average order amount.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT
    SUM(amount) AS total_sales,
    AVG(amount) AS average_order
FROM orders;
```

### Exercise 3 — Hard

Find each customer's total spending, but only show customers who spent more than 500.

**Hint:** Use `GROUP BY` and `HAVING`.

**Answer:**

```sql
SELECT
    customer,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer
HAVING SUM(amount) > 500;
```

---

## FAQ

**Q: What's the difference between `COUNT(*)` and `COUNT(column)`?**

`COUNT(*)` counts rows. `COUNT(column)` counts non-`NULL` values in that column.

```sql
SELECT COUNT(*) FROM users;
SELECT COUNT(phone) FROM users;
```

**Q: Can I use `WHERE` with aggregation?**

Yes, but `WHERE` filters rows **before** aggregation.

```sql
SELECT SUM(amount)
FROM orders
WHERE status = 'completed';
```

**Q: What's `GROUP BY` used for?**

It creates groups so you can calculate an aggregate for each group.

```sql
SELECT customer, SUM(amount)
FROM orders
GROUP BY customer;
```

**Q: What's the difference between `WHERE` and `HAVING`?**

`WHERE` filters individual rows.

`HAVING` filters groups after aggregation.

```sql
WHERE amount > 100
```

versus:

```sql
HAVING SUM(amount) > 500
```

**Q: Can I use multiple aggregate functions together?**

Yes:

```sql
SELECT
    COUNT(*) AS orders,
    SUM(amount) AS sales,
    AVG(amount) AS average,
    MAX(amount) AS largest
FROM orders;
```

**Q: What happens to NULL values?**

Most aggregate functions ignore `NULL` values.

For example:

```sql
AVG(salary)
```

doesn't include `NULL` salaries in the calculation.

`COUNT(*)` is different because it counts rows regardless of whether individual columns contain `NULL`.

---

## Key Takeaways

* 🔹 Aggregation summarizes **multiple rows into values**.
* 🔹 Main functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.
* 🔹 `GROUP BY` lets you calculate aggregates **per group**.
* 🔹 `WHERE` filters rows before aggregation.
* 🔹 `HAVING` filters groups after aggregation.

---

## Next Steps

The natural order from here is:

`GROUP BY` → create groups for aggregation
`HAVING` → filter aggregated groups
`ORDER BY` → sort aggregated results
`COUNT / SUM / AVG` → practice common business queries


## PRACTICE:

Let's practice in levels. Don't look for the answers yet—write the SQL yourself.

Use this table:

```sql
orders
-----------------------------------------
id | customer | city     | amount
-----------------------------------------
1  | Ali      | Lahore   | 500
2  | Sara     | Karachi  | 800
3  | Ahmed    | Lahore   | 300
4  | Hamza    | Karachi  | 700
5  | Ali      | Lahore   | 900
6  | Sara     | Karachi  | 400
7  | Ahmed    | Lahore   | 600
8  | Hamza    | Karachi  | 200
```

### Level 1 — Basic aggregates

1. Find the total number of orders.

2. Find the total amount of all orders.

3. Find the average order amount.

4. Find the minimum order amount.

5. Find the maximum order amount.

6. Find how many orders have an amount greater than 500.

### Level 2 — GROUP BY

7. Find the total sales for each city.

8. Find the number of orders for each city.

9. Find the average order amount for each city.

10. Find the minimum order amount for each city.

11. Find the maximum order amount for each city.

12. Find the total sales for each customer.

### Level 3 — WHERE + aggregation

13. Find the total sales from Lahore only.

14. Find the average order amount for Karachi.

15. Count the orders from Lahore where the amount is greater than 500.

16. Find the maximum order amount from Lahore.

### Level 4 — HAVING

17. Find cities whose total sales are greater than 2,000.

18. Find cities that have more than 3 orders.

19. Find customers whose total purchases are greater than 1,000.

20. Find customers whose average order amount is greater than 500.

### Level 5 — Subqueries + aggregation

21. Find the customer who made the order with the minimum amount.

22. Find the customer who made the order with the maximum amount.

23. Find all orders whose amount is greater than the average order amount.

24. Find the customers whose total purchases are greater than the overall average order amount.

### Level 6 — Challenge

25. Find the city with the highest total sales.

26. Find the customer with the highest total purchases.

27. Find the city with the highest average order amount.

28. Find the second-highest order amount.

29. Find each customer's total purchases and sort them from highest to lowest.

30. Find each city’s total sales, average order amount, minimum amount, maximum amount, and number of orders in one query.

A good progression is:

`COUNT/SUM/AVG/MIN/MAX → GROUP BY → WHERE + GROUP BY → HAVING → subqueries → advanced aggregation`

Start with **1–6**. Send me your queries, and I'll check them like an interview—no mercy for the little mistakes.
