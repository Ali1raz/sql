# CTE — Common Table Expression
## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Syntax](#syntax)
- [Beginner — Simple CTE](#beginner-simple-cte)
- [Intermediate — CTE + Aggregation](#intermediate-cte-aggregation)
- [Intermediate — CTE + JOIN](#intermediate-cte-join)
- [Advanced — Multiple CTEs](#advanced-multiple-ctes)
- [Mistake #1 — Forgetting the SELECT after the CTE](#mistake-1-forgetting-the-select-after-the-cte)
- [Mistake #2 — Forgetting the CTE name](#mistake-2-forgetting-the-cte-name)
- [Mistake #3 — Using semicolon too early](#mistake-3-using-semicolon-too-early)
- [Mistake #4 — Assuming a CTE is a permanent table](#mistake-4-assuming-a-cte-is-a-permanent-table)


## Quick Definition

A **CTE (Common Table Expression)** is a temporary named result set that you create with `WITH` and use inside a SQL query.

Think of it as:

> **Build a temporary result first, give it a name, then query it.**

CTEs are mainly useful for making complex queries **easier to read, organize, and maintain**.

---

## The Basic Idea

Without a CTE, you might write a subquery like this:

```sql
SELECT *
FROM (
    SELECT customer, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer
) AS totals
WHERE total_spent > 500;
```

With a CTE:

```sql
WITH totals AS (
    SELECT
        customer,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer
)
SELECT *
FROM totals
WHERE total_spent > 500;
```

Much easier to read.

Think:

```text
WITH totals AS (...)
        ↓
Create temporary result
        ↓
SELECT FROM totals
```

The CTE exists only for that query. It is **not a permanent table**.

---

## Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ...
FROM cte_name;
```

Example:

```sql
WITH pakistan_users AS (
    SELECT *
    FROM users
    WHERE country = 'Pakistan'
)
SELECT *
FROM pakistan_users;
```

Breaking it down:

* `WITH` → starts the CTE
* `pakistan_users` → name of the CTE
* `AS` → defines what the CTE contains
* `(SELECT ...)` → query that creates the temporary result
* Outer `SELECT` → uses the CTE

---

# Examples

## Beginner — Simple CTE

Find Pakistani users:

```sql
WITH pakistan_users AS (
    SELECT *
    FROM users
    WHERE country = 'Pakistan'
)
SELECT *
FROM pakistan_users;
```

The CTE produces:

```text
pakistan_users
       ↓
users where country = Pakistan
```

Then the outer query reads from it.

You could have written the query without a CTE:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

So for a simple query, a CTE isn't necessary.

The real benefit appears when the query gets more complicated.

---

## Intermediate — CTE + Aggregation

Find customers who spent more than 500:

```sql
WITH customer_totals AS (
    SELECT
        customer,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer
)
SELECT
    customer,
    total_spent
FROM customer_totals
WHERE total_spent > 500;
```

The process:

```text
orders
   ↓
GROUP BY customer
   ↓
SUM(amount)
   ↓
customer_totals
   ↓
WHERE total_spent > 500
```

This is one of the most useful CTE patterns.

---

## Intermediate — CTE + JOIN

Suppose you want to calculate each customer's spending and then connect it to the `users` table.

```sql
WITH customer_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT
    u.name,
    ct.total_spent
FROM users u
JOIN customer_totals ct
    ON u.id = ct.user_id;
```

Result:

| name | total_spent |
| ---- | ----------: |
| Ali  |         750 |
| Sara |        1200 |
| John |         450 |

The CTE handles the aggregation first.

The outer query handles the relationship with `users`.

---

## Advanced — Multiple CTEs

You can define multiple CTEs in one query.

Separate them with commas:

```sql
WITH customer_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
),
high_value_customers AS (
    SELECT
        user_id,
        total_spent
    FROM customer_totals
    WHERE total_spent > 1000
)
SELECT
    u.name,
    hvc.total_spent
FROM users u
JOIN high_value_customers hvc
    ON u.id = hvc.user_id;
```

Think of it as a pipeline:

```text
orders
   ↓
customer_totals
   ↓
high_value_customers
   ↓
JOIN users
   ↓
final result
```

This is where CTEs become extremely useful.

---

# CTE vs Subquery

A subquery:

```sql
SELECT *
FROM (
    SELECT
        customer,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer
) AS totals
WHERE total > 500;
```

CTE:

```sql
WITH totals AS (
    SELECT
        customer,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer
)
SELECT *
FROM totals
WHERE total > 500;
```

They can accomplish the same thing.

The main difference is **readability and organization**.

A CTE gives the intermediate result a clear name.

```text
Subquery:
"some query inside another query"

CTE:
"here's a named step in my query"
```

---

# Common Mistakes

## Mistake #1 — Forgetting the SELECT after the CTE

Wrong:

```sql
WITH totals AS (
    SELECT customer, SUM(amount)
    FROM orders
    GROUP BY customer
);
```

A CTE must be followed by a query that uses it.

Correct:

```sql
WITH totals AS (
    SELECT
        customer,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer
)
SELECT *
FROM totals;
```

---

## Mistake #2 — Forgetting the CTE name

Wrong:

```sql
WITH AS (
    SELECT *
    FROM users
)
SELECT *
FROM users;
```

Correct:

```sql
WITH pakistan_users AS (
    SELECT *
    FROM users
    WHERE country = 'Pakistan'
)
SELECT *
FROM pakistan_users;
```

---

## Mistake #3 — Using semicolon too early

Wrong:

```sql
WITH totals AS (
    SELECT customer, SUM(amount)
    FROM orders
    GROUP BY customer
);
SELECT *
FROM totals;
```

The semicolon ends the statement.

Correct:

```sql
WITH totals AS (
    SELECT
        customer,
        SUM(amount) AS total
    FROM orders
    GROUP BY customer
)
SELECT *
FROM totals;
```

Put the semicolon at the end.

---

## Mistake #4 — Assuming a CTE is a permanent table

This:

```sql
WITH totals AS (
    SELECT ...
)
SELECT *
FROM totals;
```

does **not** create a permanent database table.

After the query finishes, the CTE is gone.

If you need permanent storage, you'd use something like:

```sql
CREATE TABLE ...
```

---

# Performance Notes

A CTE is primarily a **query organization tool**. Don't assume that using a CTE automatically makes a query faster.

Depending on the database system and query, a CTE may be optimized, inlined, or materialized differently.

For example:

```sql
WITH totals AS (
    SELECT
        user_id,
        SUM(amount) AS total
    FROM orders
    GROUP BY user_id
)
SELECT *
FROM totals
WHERE total > 1000;
```

The performance depends on the underlying query, indexes, database engine, and execution plan.

For performance-sensitive queries, check the execution plan rather than assuming:

> CTE = faster

The main beginner benefit of CTEs is **cleaner SQL**.

---

# Practice

### Exercise 1 — Easy

Create a CTE containing users from Pakistan and select from it.

```sql
-- Your query here
```

**Answer:**

```sql
WITH pakistan_users AS (
    SELECT *
    FROM users
    WHERE country = 'Pakistan'
)
SELECT *
FROM pakistan_users;
```

### Exercise 2 — Medium

Calculate total spending per customer, then return only customers who spent more than 1000.

**Answer:**

```sql
WITH customer_totals AS (
    SELECT
        customer,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer
)
SELECT *
FROM customer_totals
WHERE total_spent > 1000;
```

### Exercise 3 — Hard

Calculate each user's total spending and show their name.

**Answer:**

```sql
WITH user_totals AS (
    SELECT
        user_id,
        SUM(amount) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT
    u.name,
    ut.total_spent
FROM users u
JOIN user_totals ut
    ON u.id = ut.user_id;
```

---

# FAQ

**Q: What does CTE stand for?**

Common Table Expression.

It creates a temporary named result that can be used by the following query.

**Q: Does a CTE create a table?**

No.

It only exists for the duration of that SQL statement.

**Q: Why use a CTE instead of a subquery?**

Mainly for readability.

Compare:

```sql
WITH totals AS (...)
SELECT *
FROM totals;
```

with a deeply nested subquery. CTEs make complex queries easier to break into logical steps.

**Q: Can I have multiple CTEs?**

Yes:

```sql
WITH users_data AS (
    SELECT ...
),
orders_data AS (
    SELECT ...
)
SELECT ...
FROM users_data
JOIN orders_data
    ON ...;
```

**Q: Can a CTE use another CTE?**

Yes:

```sql
WITH totals AS (
    SELECT ...
),
large_totals AS (
    SELECT *
    FROM totals
    WHERE total > 1000
)
SELECT *
FROM large_totals;
```

The second CTE can use the first one.

**Q: Is a CTE faster than a subquery?**

Not necessarily.

Don't choose a CTE because you assume it will improve performance. Choose it when it makes the query clearer, then check the execution plan if performance matters.

**Q: Can I use JOIN with a CTE?**

Absolutely:

```sql
WITH totals AS (
    SELECT user_id, SUM(amount) AS total
    FROM orders
    GROUP BY user_id
)
SELECT u.name, t.total
FROM users u
JOIN totals t
    ON u.id = t.user_id;
```

---

# Key Takeaways

* 🔹 `CTE` means **Common Table Expression**.
* 🔹 Start one with `WITH`.
* 🔹 A CTE is a **named temporary result**.
* 🔹 CTEs make complex queries easier to read and break into steps.
* 🔹 You can create multiple CTEs and have one CTE use another.
* 🔹 CTEs don't automatically make queries faster.

---

# Next Steps

Your SQL progression is now getting into intermediate territory:

`Subqueries` → query inside query
`CTEs` → named query steps
`Window Functions` → calculations across rows without collapsing them
`CASE` → conditional logic
`UNION` → combine results from multiple queries
