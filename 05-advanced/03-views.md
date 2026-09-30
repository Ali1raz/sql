# SQL VIEWS
## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Why Views Matter](#why-views-matter)
- [Syntax](#syntax)
- [Beginner Example](#beginner-example)
- [Intermediate Example](#intermediate-example)
- [Advanced Example](#advanced-example)


## Quick Definition

A **view** is a virtual table created from a `SELECT` query. It usually does not store the actual data itself; instead, when you query the view, the database runs the underlying query.

Think of a view as a **saved SQL query that you can treat like a table**.

---

## The Basic Idea

Suppose you have:

```sql
users
```

| id | name | country  | status   |
| -: | ---- | -------- | -------- |
|  1 | Ali  | Pakistan | active   |
|  2 | Sara | UAE      | active   |
|  3 | John | USA      | inactive |

You frequently need only active users.

Instead of repeatedly writing:

```sql
SELECT id, name, country
FROM users
WHERE status = 'active';
```

Create a view:

```sql
CREATE VIEW active_users AS
SELECT id, name, country
FROM users
WHERE status = 'active';
```

Now you can simply:

```sql
SELECT *
FROM active_users;
```

Mental model:

```text
Base tables
    ↓
SELECT query
    ↓
   VIEW
    ↓
Query the view like a table
```

---

## Why Views Matter

Views are useful when you want to:

* Simplify complicated queries
* Reuse the same query logic
* Hide unnecessary columns
* Provide controlled access to data
* Create a clean interface for applications or reports

For example, instead of giving an application access to every column in `users`, you can create a view containing only safe columns.

---

## Syntax

### Create a View

```sql
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;
```

Example:

```sql
CREATE VIEW active_users AS
SELECT id, name, country
FROM users
WHERE status = 'active';
```

### Query a View

```sql
SELECT *
FROM active_users;
```

### Delete a View

```sql
DROP VIEW active_users;
```

Some databases support:

```sql
DROP VIEW IF EXISTS active_users;
```

---

# Examples

## Beginner Example

Create a view containing Pakistani users:

```sql
CREATE VIEW pakistan_users AS
SELECT id, name, country
FROM users
WHERE country = 'Pakistan';
```

Query it:

```sql
SELECT *
FROM pakistan_users;
```

Expected result:

| id | name | country  |
| -: | ---- | -------- |
|  1 | Ali  | Pakistan |

The view behaves like a table when you query it.

---

## Intermediate Example

Suppose you have:

```sql
orders
```

| id | customer_id | amount | status    |
| -: | ----------: | -----: | --------- |
|  1 |         101 |    500 | completed |
|  2 |         102 |    300 | pending   |
|  3 |         101 |    700 | completed |

Create a view containing completed orders:

```sql
CREATE VIEW completed_orders AS
SELECT id, customer_id, amount
FROM orders
WHERE status = 'completed';
```

Now:

```sql
SELECT *
FROM completed_orders
ORDER BY amount DESC;
```

Result:

| id | customer_id | amount |
| -: | ----------: | -----: |
|  3 |         101 |    700 |
|  1 |         101 |    500 |

The advantage is that the filtering logic is already saved.

---

## Advanced Example

Views can contain joins and aggregations.

Suppose:

```text
users
  ↓
orders
```

Create a customer summary:

```sql
CREATE VIEW customer_order_summary AS
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS total_orders,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o
    ON u.id = o.customer_id
GROUP BY u.id, u.name;
```

Now:

```sql
SELECT *
FROM customer_order_summary;
```

Possible result:

| id | name | total_orders | total_spent |
| -: | ---- | -----------: | ----------: |
|  1 | Ali  |            5 |        4500 |
|  2 | Sara |            2 |        1200 |
|  3 | John |            0 |           0 |

Instead of rewriting that entire join and aggregation every time, you query the view.

---

# Common Mistakes

### Mistake #1: Thinking a view is a physical copy of the data

```sql
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE status = 'active';
```

A normal view generally **doesn't store a separate copy of the rows**.

If the underlying `users` data changes, the view reflects those changes when queried.

---

### Mistake #2: Treating a view like a backup

A view is not a backup.

```sql
DROP VIEW active_users;
```

This removes the view definition, not the underlying `users` data.

---

### Mistake #3: Creating views for everything

You don't need a view for every simple query.

This:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

probably doesn't need its own view unless the query is reused or serves a specific purpose.

---

### Mistake #4: Forgetting that complex views can still be expensive

This view:

```sql
CREATE VIEW huge_report AS
SELECT ...
FROM orders
JOIN users ...
JOIN products ...
GROUP BY ...;
```

doesn't magically make the underlying query faster.

When you run:

```sql
SELECT *
FROM huge_report;
```

the database still has to process the underlying logic.

---

# Performance Notes

A normal view is mainly about **organization, abstraction, and security**, not automatically performance.

For example:

```sql
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE status = 'active';
```

Then:

```sql
SELECT *
FROM active_users;
```

The database still needs to execute the underlying query.

Indexes on the underlying tables can still help:

```sql
CREATE INDEX idx_users_status
ON users(status);
```

For very expensive queries that you want to physically store and refresh, some databases provide **materialized views**.

Basic difference:

```text
View
→ Stores query definition
→ Data comes from underlying tables

Materialized View
→ Stores query result
→ Usually needs refreshing
→ Can improve performance for expensive reports
```

Materialized-view syntax and behavior are database-specific.

---

# Practice

### Exercise 1 — Easy

Create a view containing users whose status is `active`.

Hint:

```sql
CREATE VIEW ...
AS
SELECT ...
FROM users
WHERE ...;
```

Solution:

```sql
CREATE VIEW active_users AS
SELECT id, name, country
FROM users
WHERE status = 'active';
```

---

### Exercise 2 — Medium

Create a view containing orders greater than `1000`.

Solution:

```sql
CREATE VIEW large_orders AS
SELECT id, customer_id, amount
FROM orders
WHERE amount > 1000;
```

---

### Exercise 3 — Hard

Create a view showing each customer's total number of orders and total amount spent.

Hint: use:

```sql
JOIN
COUNT()
SUM()
GROUP BY
```

Solution:

```sql
CREATE VIEW customer_summary AS
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS total_orders,
    COALESCE(SUM(o.amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o
    ON u.id = o.customer_id
GROUP BY u.id, u.name;
```

---

# FAQ

**Q: Is a view a table?**

A: It behaves like a table when you query it, but a normal view is a saved query rather than an independent copy of the data.

```sql
SELECT *
FROM active_users;
```

**When to use:** When you want a reusable query interface.

---

**Q: Does a view store data?**

A: A normal view generally stores the **query definition**, not the query result.

```text
View → Query definition
Materialized View → Stored result
```

---

**Q: Can I use `WHERE` with a view?**

Yes.

```sql
SELECT *
FROM active_users
WHERE country = 'Pakistan';
```

You can apply additional filtering when querying the view.

---

**Q: Can a view use JOINs?**

Yes.

```sql
CREATE VIEW customer_orders AS
SELECT u.name, o.amount
FROM users u
JOIN orders o
    ON u.id = o.customer_id;
```

---

**Q: Can I update data through a view?**

Sometimes.

Simple views may be updatable, but views involving things such as `GROUP BY`, aggregation, `DISTINCT`, or complicated joins often cannot be updated directly. Exact rules depend on the database.

---

**Q: What's the difference between a view and a CTE?**

A **CTE** normally exists only for one SQL statement:

```sql
WITH active_users AS (
    SELECT *
    FROM users
    WHERE status = 'active'
)
SELECT *
FROM active_users;
```

A **view** is saved in the database and can be reused:

```sql
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE status = 'active';
```

---

**Q: Does a view make a query faster?**

Not necessarily.

A normal view mainly improves **reusability, readability, and abstraction**. Performance depends on the underlying query and database optimizer.

---

**Q: Can I delete a view without deleting the table?**

Yes.

```sql
DROP VIEW active_users;
```

The underlying `users` table remains.

---

# Key Takeaways

* 🔹 A **view is a saved `SELECT` query** that can be queried like a table.
* 🔹 Views help with **reuse, simplicity, security, and abstraction**.
* 🔹 A normal view usually **doesn't store a separate copy of the data**.
* 🔹 Views don't automatically make queries faster.
* 🔹 Complex views can contain `JOIN`, `GROUP BY`, aggregates, and other SQL logic.

# Next Steps

After views, the natural concepts to learn are:

`Views` → `Materialized Views` → `Transactions` → `ACID` → `Stored Procedures` → `Triggers`
