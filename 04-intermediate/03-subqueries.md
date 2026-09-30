# SQL SUBQUERIES
## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Syntax](#syntax)
- [Beginner — Subquery Returning One Value](#beginner-subquery-returning-one-value)
- [Intermediate — Subquery with IN](#intermediate-subquery-with-in)
- [Intermediate — Subquery with EXISTS](#intermediate-subquery-with-exists)
- [Advanced — Correlated Subquery](#advanced-correlated-subquery)
- [Mistake #1 — Subquery returns multiple rows when one value is expected](#mistake-1-subquery-returns-multiple-rows-when-one-value-is-expected)
- [Mistake #2 — Confusing IN and EXISTS](#mistake-2-confusing-in-and-exists)
- [Mistake #3 — Forgetting the relationship in a correlated subquery](#mistake-3-forgetting-the-relationship-in-a-correlated-subquery)


## Quick Definition

A **subquery** is a SQL query written **inside another SQL query**.

Think of it as:

> **Query inside a query.**

The inner query produces a result, and the outer query uses that result.

For example, first find the average order amount, then find orders above that average:

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

The inner query:

```sql
SELECT AVG(amount)
FROM orders;
```

runs first and produces a value.

The outer query then uses that value.

---

## The Basic Idea

Suppose we have:

| id | customer | amount |
| -: | -------- | -----: |
|  1 | Ali      |    100 |
|  2 | Sara     |    500 |
|  3 | John     |    200 |
|  4 | Ahmed    |    800 |

Average = `400`

Now:

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

Result:

| id | customer | amount |
| -: | -------- | -----: |
|  2 | Sara     |    500 |
|  4 | Ahmed    |    800 |

The mental model:

```text
Inner query
     ↓
Calculate average
     ↓
400
     ↓
Outer query
     ↓
Find orders > 400
```

---

## Syntax

The general structure is:

```sql
SELECT columns
FROM table
WHERE column operator (
    SELECT column
    FROM table
    WHERE condition
);
```

The important part is:

```sql
(
    SELECT ...
)
```

The parentheses tell SQL that this is a subquery.

---

# Examples

## Beginner — Subquery Returning One Value

Find orders above the average order amount:

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

Here the subquery returns **one value**:

```text
AVG(amount) → 425
```

The outer query compares every order against `425`.

---

## Intermediate — Subquery with IN

Suppose:

`users`

| id | name | country  |
| -: | ---- | -------- |
|  1 | Ali  | Pakistan |
|  2 | Sara | USA      |
|  3 | John | Pakistan |

`orders`

|  id | user_id | amount |
| --: | ------: | -----: |
| 101 |       1 |    500 |
| 102 |       2 |    300 |
| 103 |       3 |    700 |

Find orders belonging to users from Pakistan:

```sql
SELECT *
FROM orders
WHERE user_id IN (
    SELECT id
    FROM users
    WHERE country = 'Pakistan'
);
```

The inner query returns:

```text
1
3
```

Then the outer query effectively searches:

```sql
WHERE user_id IN (1, 3)
```

Result:

|  id | user_id | amount |
| --: | ------: | -----: |
| 101 |       1 |    500 |
| 103 |       3 |    700 |

---

## Intermediate — Subquery with EXISTS

`EXISTS` checks whether the subquery returns **at least one row**.

Find users who have placed an order:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

For each user, SQL asks:

> “Does an order exist for this user?”

If yes → include the user.

If no → exclude them.

This is especially useful when you care about **whether related data exists**, rather than needing the related data itself.

---

## Advanced — Correlated Subquery

A correlated subquery references a column from the outer query.

Example: find users whose order amount is greater than their own average order amount:

```sql
SELECT *
FROM orders o
WHERE amount > (
    SELECT AVG(amount)
    FROM orders o2
    WHERE o2.user_id = o.user_id
);
```

The inner query depends on the current row of the outer query.

Conceptually:

```text
Ali's order
   ↓
Calculate Ali's average
   ↓
Compare order against Ali's average

Sara's order
   ↓
Calculate Sara's average
   ↓
Compare order against Sara's average
```

This is powerful, but potentially more expensive than simpler approaches.

---

# Subquery Types

There are several ways you can use subqueries.

### 1. Scalar subquery

Returns a single value.

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

### 2. Multi-row subquery

Returns multiple values.

Usually used with `IN`:

```sql
SELECT *
FROM orders
WHERE user_id IN (
    SELECT id
    FROM users
    WHERE country = 'Pakistan'
);
```

### 3. EXISTS subquery

Checks whether rows exist:

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

### 4. Correlated subquery

Depends on the outer query:

```sql
SELECT *
FROM orders o
WHERE amount > (
    SELECT AVG(amount)
    FROM orders o2
    WHERE o2.user_id = o.user_id
);
```

---

# Common Mistakes

## Mistake #1 — Subquery returns multiple rows when one value is expected

This can be wrong:

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT amount
    FROM orders
);
```

The inner query returns many amounts, but `>` expects a single value in this form.

Use `IN` if you actually want to compare against multiple values:

```sql
SELECT *
FROM orders
WHERE amount IN (
    SELECT amount
    FROM orders
);
```

Or use an aggregate if you need one value:

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

---

## Mistake #2 — Confusing IN and EXISTS

`IN` compares against values:

```sql
WHERE user_id IN (
    SELECT id
    FROM users
);
```

`EXISTS` checks whether a matching row exists:

```sql
WHERE EXISTS (
    SELECT 1
    FROM users
    WHERE users.id = orders.user_id
);
```

Both can solve similar problems, but they express different logic.

---

## Mistake #3 — Forgetting the relationship in a correlated subquery

Bad:

```sql
SELECT *
FROM orders o
WHERE amount > (
    SELECT AVG(amount)
    FROM orders o2
);
```

This compares against the **overall average**.

If you want each user's average, you need:

```sql
SELECT *
FROM orders o
WHERE amount > (
    SELECT AVG(amount)
    FROM orders o2
    WHERE o2.user_id = o.user_id
);
```

That `WHERE` connects the inner query to the current outer row.

---

# Performance Notes

Subqueries aren't automatically slow.

A simple scalar subquery such as:

```sql
SELECT AVG(amount)
FROM orders;
```

is usually straightforward for the database to execute.

However, **correlated subqueries** can become expensive because the inner query may need to be evaluated repeatedly for outer rows.

Sometimes a `JOIN` or `GROUP BY` provides a more efficient alternative.

For example, instead of repeatedly calculating customer totals with a correlated subquery, you might use:

```sql
SELECT
    u.id,
    u.name,
    SUM(o.amount) AS total_spent
FROM users u
JOIN orders o
    ON u.id = o.user_id
GROUP BY u.id, u.name;
```

The important rule:

> Don't avoid subqueries just because they're subqueries. Check the query plan when performance matters.

---

# Practice

### Exercise 1 — Easy

Find products that cost more than the average product price.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

### Exercise 2 — Medium

Find orders belonging to users from Pakistan.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT *
FROM orders
WHERE user_id IN (
    SELECT id
    FROM users
    WHERE country = 'Pakistan'
);
```

### Exercise 3 — Hard

Find users who have at least one order.

**Hint:** Try `EXISTS`.

**Answer:**

```sql
SELECT *
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

---

# FAQ

**Q: What is a subquery?**

A query inside another query.

```sql
SELECT *
FROM orders
WHERE amount > (
    SELECT AVG(amount)
    FROM orders
);
```

**Q: When should I use a subquery instead of a JOIN?**

Use a subquery when the logic naturally reads as:

> “Find rows based on the result of another query.”

Use a `JOIN` when you need to combine columns/data from related tables.

**Q: Can a subquery return multiple rows?**

Yes, but the outer operation must support multiple values.

For example:

```sql
WHERE user_id IN (
    SELECT id
    FROM users
);
```

`IN` can handle multiple results.

**Q: What's the difference between `IN` and `EXISTS`?**

`IN` compares a value against a list of results.

`EXISTS` checks whether at least one matching row exists.

**Q: What is a correlated subquery?**

A subquery that references a column from the outer query.

```sql
WHERE o2.user_id = o.user_id
```

**Q: Can I put a subquery inside SELECT?**

Yes:

```sql
SELECT
    name,
    (SELECT COUNT(*)
     FROM orders o
     WHERE o.user_id = u.id) AS order_count
FROM users u;
```

**Q: Can I use a subquery inside FROM?**

Yes. The result acts like a temporary table:

```sql
SELECT *
FROM (
    SELECT customer, SUM(amount) AS total
    FROM orders
    GROUP BY customer
) AS totals;
```

---

# Key Takeaways

* 🔹 A **subquery is a query inside another query**.
* 🔹 Scalar subqueries return one value.
* 🔹 `IN` works well when the subquery returns multiple values.
* 🔹 `EXISTS` checks whether matching rows exist.
* 🔹 Correlated subqueries depend on the outer query and can be more expensive.
* 🔹 Sometimes a `JOIN` or `GROUP BY` is a better alternative.

---

# Next Steps

The useful progression is:

`Subqueries` → query inside query
`IN` → work with multiple subquery results
`EXISTS` → check whether related data exists
`Correlated subqueries` → outer query + inner query relationship
`CTEs (WITH)` → make complex queries easier to read
`Window functions` → advanced calculations without collapsing rows
