# SQL JOINs
## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Syntax](#syntax)
- [1. INNER JOIN](#1-inner-join)
- [Beginner — Join Users and Orders](#beginner-join-users-and-orders)
- [Intermediate — JOIN + WHERE](#intermediate-join-where)
- [Intermediate — JOIN + ORDER BY](#intermediate-join-order-by)
- [Advanced — JOIN + GROUP BY](#advanced-join-group-by)
- [Mistake #1 — Forgetting the ON condition](#mistake-1-forgetting-the-on-condition)
- [Mistake #2 — Joining the wrong columns](#mistake-2-joining-the-wrong-columns)
- [Mistake #3 — Accidentally turning LEFT JOIN into INNER JOIN](#mistake-3-accidentally-turning-left-join-into-inner-join)
- [Mistake #4 — Getting duplicate rows](#mistake-4-getting-duplicate-rows)
- [Exercise 1 — Easy](#exercise-1-easy)
- [Exercise 2 — Medium](#exercise-2-medium)
- [Exercise 3 — Hard](#exercise-3-hard)


## Quick Definition

A `JOIN` combines rows from **two or more tables** using a related column.

The main idea:

> **JOIN = connect related data from different tables.**

For example, instead of storing the customer's name inside every order, you might have:

```text
users
+----+-------+
| id | name  |
+----+-------+
| 1  | Ali   |
| 2  | Sara  |
+----+-------+

orders
+----+---------+--------+
| id | user_id | amount |
+----+---------+--------+
| 101| 1       | 500    |
| 102| 2       | 300    |
+----+---------+--------+
```

`orders.user_id` connects to `users.id`.

---

## The Basic Idea

A simple `JOIN`:

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

Result:

| name | amount |
| ---- | -----: |
| Ali  |    500 |
| Sara |    300 |

The database matches:

```text
users.id = orders.user_id
```

Think of it like matching two lists using a common ID.

---

## Syntax

```sql
SELECT columns
FROM table1
JOIN table2
    ON table1.column = table2.column;
```

Example:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

Breaking it down:

* `SELECT` → columns you want
* `FROM users` → starting table
* `JOIN orders` → connect the orders table
* `ON` → tells SQL how the tables are related
* `users.id = orders.user_id` → matching condition

---

# Types of JOINs

The important ones to learn first are:

```text
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
```

There's also `CROSS JOIN`, but don't worry about it until the basics are solid.

---

## 1. INNER JOIN

Returns only rows that have a match in **both tables**.

```sql
SELECT
    users.name,
    orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

You can also simply write:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

`JOIN` normally means `INNER JOIN`.

### Example

Users:

| id | name |
| -: | ---- |
|  1 | Ali  |
|  2 | Sara |
|  3 | John |

Orders:

|  id | user_id | amount |
| --: | ------: | -----: |
| 101 |       1 |    500 |
| 102 |       2 |    300 |

John has no order.

With `INNER JOIN`:

| name | amount |
| ---- | -----: |
| Ali  |    500 |
| Sara |    300 |

John disappears because there is no matching order.

---

# 2. LEFT JOIN

Returns **all rows from the left table**, even if there is no match in the right table.

```sql
SELECT
    users.name,
    orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

Result:

| name | amount |
| ---- | -----: |
| Ali  |    500 |
| Sara |    300 |
| John |   NULL |

John is included because `users` is the left table.

There simply isn't a matching order, so `orders.amount` becomes `NULL`.

### When is this useful?

Finding users who **have or don't have orders**.

For example:

```sql
SELECT
    users.name,
    orders.id AS order_id
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

---

# 3. RIGHT JOIN

`RIGHT JOIN` is basically the opposite of `LEFT JOIN`.

It returns **all rows from the right table**.

```sql
SELECT
    users.name,
    orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

In practice, `LEFT JOIN` is often easier to reason about because you can simply put the table you want to preserve on the left.

---

# 4. FULL OUTER JOIN

Returns rows from **both tables**, whether or not they have a match.

```sql
SELECT
    users.name,
    orders.amount
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

Conceptually:

```text
Matching rows       → included
User without order  → included
Order without user  → included
```

Not every database supports `FULL OUTER JOIN` directly.

---

# Examples

## Beginner — Join Users and Orders

Get each user's name and their order amount:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

Result:

| name | amount |
| ---- | -----: |
| Ali  |    500 |
| Sara |    300 |

---

## Intermediate — JOIN + WHERE

Find orders from users in Pakistan:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id
WHERE users.country = 'Pakistan';
```

Here:

```text
JOIN  → connects users and orders
WHERE → filters the connected rows
```

---

## Intermediate — JOIN + ORDER BY

Show orders from highest amount to lowest:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id
ORDER BY orders.amount DESC;
```

---

## Advanced — JOIN + GROUP BY

Find the total amount spent by each user:

```sql
SELECT
    users.name,
    SUM(orders.amount) AS total_spent
FROM users
JOIN orders
    ON users.id = orders.user_id
GROUP BY users.id, users.name;
```

Example:

| name | total_spent |
| ---- | ----------: |
| Ali  |         750 |
| Sara |         500 |

This is a very common real-world pattern:

```text
JOIN
  ↓
connect related data
  ↓
GROUP BY
  ↓
create groups
  ↓
SUM()
  ↓
calculate totals
```

---

# Using Aliases

Long table names can make queries ugly.

Instead of:

```sql
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

Use aliases:

```sql
SELECT
    u.name,
    o.amount
FROM users AS u
JOIN orders AS o
    ON u.id = o.user_id;
```

This is extremely common in real SQL.

You can even omit `AS`:

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

---

# Common Mistakes

## Mistake #1 — Forgetting the ON condition

Bad:

```sql
SELECT *
FROM users
JOIN orders;
```

You haven't told SQL how the tables should be connected.

Correct:

```sql
SELECT *
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

---

## Mistake #2 — Joining the wrong columns

Wrong:

```sql
ON users.id = orders.id
```

Usually you want:

```sql
ON users.id = orders.user_id
```

The exact relationship depends on your schema.

---

## Mistake #3 — Accidentally turning LEFT JOIN into INNER JOIN

Consider:

```sql
SELECT
    u.name,
    o.amount
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
WHERE o.amount > 100;
```

The `WHERE` condition removes rows where `o.amount` is `NULL`.

So users without orders disappear.

If your intention is to preserve users without orders, be careful about where you put filtering conditions.

---

## Mistake #4 — Getting duplicate rows

Suppose Ali has three orders.

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Ali will appear **three times**.

That's not necessarily a mistake.

One user can have many orders:

```text
Ali
 ├── Order 101
 ├── Order 102
 └── Order 103
```

If you want one row per user, aggregation may be what you need:

```sql
SELECT
    u.name,
    SUM(o.amount) AS total_spent
FROM users u
JOIN orders o
    ON u.id = o.user_id
GROUP BY u.id, u.name;
```

---

# Performance Notes

JOINs can become expensive when tables contain millions of rows.

The columns used for joining are especially important:

```sql
ON users.id = orders.user_id
```

Typically, `users.id` is already indexed because it's a primary key. `orders.user_id` is commonly indexed as well.

For example:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Indexes can make joins significantly more efficient, especially for large tables.

Don't add indexes blindly, though. They consume storage and can add overhead to data modifications.

---

# Practice

## Exercise 1 — Easy

Get the user name and order amount.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

---

## Exercise 2 — Medium

Find all users who have placed an order.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT DISTINCT
    u.name
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

`DISTINCT` prevents users with multiple orders from appearing multiple times.

---

## Exercise 3 — Hard

Find every user and their total spending, including users who have **never placed an order**.

**Hint:** You need `LEFT JOIN`.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT
    u.name,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id
GROUP BY u.id, u.name;
```

`COALESCE()` changes `NULL` into `0` for users with no orders.

---

# How to remember:

```
INNER → matching records
LEFT  → everything on left
RIGHT → everything on right
FULL  → everything on both
```
# FAQ

**Q: What's the difference between INNER JOIN and LEFT JOIN?**

`INNER JOIN` keeps only matching rows.

`LEFT JOIN` keeps every row from the left table, even without a match.

```sql
-- Only users with orders
SELECT *
FROM users u
INNER JOIN orders o
    ON u.id = o.user_id;
```

```sql
-- All users, even those without orders
SELECT *
FROM users u
LEFT JOIN orders o
    ON u.id = o.user_id;
```

**Q: Why do I need `ON`?**

`ON` tells SQL which columns establish the relationship between the tables.

```sql
ON u.id = o.user_id
```

**Q: Why am I getting duplicate users?**

Because one user may have multiple matching rows.

For example:

```text
Ali → Order 1
Ali → Order 2
Ali → Order 3
```

The JOIN correctly returns three rows.

Use `DISTINCT` or aggregation depending on what you actually need.

**Q: Can I JOIN more than two tables?**

Yes.

```sql
SELECT
    u.name,
    o.amount,
    p.name AS product
FROM users u
JOIN orders o
    ON u.id = o.user_id
JOIN products p
    ON o.product_id = p.id;
```

**Q: Should I use LEFT JOIN or INNER JOIN?**

It depends on whether unmatched rows matter.

If you only want records that have a match → `INNER JOIN`.

If you want everything from your main table, even without a match → `LEFT JOIN`.

**Q: What's the difference between JOIN and WHERE?**

`JOIN` connects tables.

`WHERE` filters the resulting rows.

```sql
SELECT u.name, o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id
WHERE o.amount > 500;
```

**Q: Why are aliases like `u` and `o` useful?**

They make queries shorter and clearer:

```sql
FROM users u
JOIN orders o
    ON u.id = o.user_id
```

instead of repeatedly writing full table names.

---

# Key Takeaways

* 🔹 `JOIN` connects related tables.
* 🔹 `INNER JOIN` returns matching rows only.
* 🔹 `LEFT JOIN` keeps everything from the left table.
* 🔹 `ON` defines how the tables are related.
* 🔹 One-to-many relationships naturally produce multiple rows.
* 🔹 `JOIN + GROUP BY + aggregate functions` is one of the most important SQL patterns.

---

# Next Steps

Learn the JOIN family in this order:

`INNER JOIN` → basic table relationships
`LEFT JOIN` → unmatched rows
`JOIN + WHERE` → filtering joined data
`JOIN + GROUP BY` → summaries across tables
`Multiple JOINs` → working with real database schemas
`SELF JOIN` → joining a table to itself
