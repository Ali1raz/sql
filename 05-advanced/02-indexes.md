# SQL INDEXES

## Quick Definition

An **index** is a data structure that helps the database **find rows faster** without scanning the entire table.

Think of it like the index at the back of a book.

Without an index:

```text
Database → check row 1
         → check row 2
         → check row 3
         → ...
         → find matching row
```

With an index:

```text
Database → use index → find matching rows quickly
```

The trade-off is that indexes require **extra storage** and can make `INSERT`, `UPDATE`, and `DELETE` more expensive.

---

## The Basic Idea

Suppose you have:

```sql id="u9n6r3"
SELECT *
FROM users
WHERE email = 'ali@example.com';
```

If `users` contains millions of rows and `email` isn't indexed, the database may need to inspect many rows.

Create an index:

```sql id="8s0z9m"
CREATE INDEX idx_users_email
ON users(email);
```

Now the database has an additional structure it can use to find emails more efficiently.

---

# Syntax

Create an index:

```sql id="8f9p0k"
CREATE INDEX index_name
ON table_name(column_name);
```

Example:

```sql id="f7v6qw"
CREATE INDEX idx_users_email
ON users(email);
```

Delete an index:

```sql id="2x6q0p"
DROP INDEX idx_users_email;
```

The exact `DROP INDEX` syntax varies between database systems.

---

# Examples

## Beginner — Index a Frequently Searched Column

Suppose this query runs frequently:

```sql id="h8b4z1"
SELECT *
FROM users
WHERE email = 'ali@example.com';
```

Create:

```sql id="3p9w4n"
CREATE INDEX idx_users_email
ON users(email);
```

Now the database has an index specifically for `email`.

A common rule:

> Columns frequently used in `WHERE` conditions are candidates for indexes.

But don't index every column.

---

## Intermediate — Index a Foreign Key

Suppose:

```sql id="v0xk2c"
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

`users.id` is usually already indexed because it is a primary key.

You may also want an index on:

```sql id="p7v3xm"
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

This can help queries that frequently find orders for a particular user or join `orders` through `user_id`.

---

## Intermediate — Index a Column Used for Sorting

Suppose you frequently run:

```sql id="c6v2qb"
SELECT *
FROM products
ORDER BY price;
```

An index on `price` may help some queries:

```sql id="w8k2za"
CREATE INDEX idx_products_price
ON products(price);
```

Whether it actually improves the query depends on the database, query, table size, and execution plan.

---

## Advanced — Composite Index

You can index multiple columns together:

```sql id="h9f3jk"
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

This is called a **composite index** or **multi-column index**.

It's useful for queries such as:

```sql id="j6x4yp"
SELECT *
FROM orders
WHERE customer_id = 10
  AND status = 'completed';
```

The order of columns matters.

For:

```sql id="1a8d5r"
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

the index is primarily organized around:

```text
customer_id → status
```

This can be useful for:

```sql id="x7n2pf"
WHERE customer_id = 10
```

and:

```sql id="c4r8md"
WHERE customer_id = 10
  AND status = 'completed'
```

But it may not be equally useful for:

```sql id="n3q6za"
WHERE status = 'completed'
```

This is commonly called the **leftmost-prefix principle**.

---

# Unique Indexes

A unique index prevents duplicate values.

```sql id="j8k5sx"
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

Now the database won't allow two users with the same email.

For example:

```text id="z6v3yc"
ali@example.com
ali@example.com  ← rejected
```

However, when uniqueness is part of the table's data rules, you will often define it as a `UNIQUE` constraint instead:

```sql id="k0w9fm"
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

The database can create an appropriate unique index behind the scenes, depending on the DBMS.

---

# Primary Keys and Indexes

When you create:

```sql id="r6d4qt"
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

most relational databases automatically create an index associated with the primary key.

So you usually don't need to manually create:

```sql id="b5x7vm"
CREATE INDEX idx_users_id
ON users(id);
```

The primary key is already indexed in typical systems.

---

# Common Mistakes

## Mistake #1 — Indexing Everything

Don't do this:

```sql id="m2p8za"
CREATE INDEX idx1 ON users(name);
CREATE INDEX idx2 ON users(age);
CREATE INDEX idx3 ON users(country);
CREATE INDEX idx4 ON users(phone);
CREATE INDEX idx5 ON users(created_at);
CREATE INDEX idx6 ON users(status);
```

More indexes aren't automatically better.

Every index consumes storage and can add work when rows are inserted, updated, or deleted.

Create indexes based on actual query patterns.

---

## Mistake #2 — Ignoring Composite Index Order

These are not necessarily equivalent:

```sql id="4g7w1p"
CREATE INDEX idx1
ON orders(customer_id, status);
```

and:

```sql id="8j2s6v"
CREATE INDEX idx2
ON orders(status, customer_id);
```

The column order can affect which queries can efficiently use the index.

Design the index around the queries you actually run.

---

## Mistake #3 — Assuming an Index Guarantees Faster Queries

An index doesn't guarantee that the database will use it.

For example:

```sql id="j7k4wq"
SELECT *
FROM users
WHERE country = 'Pakistan';
```

If 90% of the table is from Pakistan, the optimizer may decide that scanning the table is cheaper than using the index.

The database's optimizer makes the decision.

---

## Mistake #4 — Functions Can Prevent Efficient Index Usage

For example:

```sql id="r0y4zc"
SELECT *
FROM users
WHERE LOWER(email) = 'ali@example.com';
```

Depending on the database and available indexes, a normal index on `email` may not be useful for this expression.

Some databases support expression/function-based indexes.

Another option is storing/searching normalized values appropriately.

The exact solution depends on the database system.

---

# Performance Notes

Indexes generally improve **read performance** but have costs.

Think of the trade-off:

```text id="7n8q2k"
More indexes
    ↓
Faster potential reads
    ↓
More storage
    +
More work during INSERT/UPDATE/DELETE
```

Indexes are especially useful for columns frequently used in:

```sql id="k3y6fw"
WHERE
JOIN
ORDER BY
GROUP BY
```

But the best index depends on the actual query.

For serious performance work, use your database's execution-plan tools, such as:

```sql id="q8r2mw"
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 10;
```

Some databases also support variants such as `EXPLAIN ANALYZE`.

---

# Index Selectivity

A useful concept is **selectivity**.

Suppose you have one million users.

This column:

```text
gender
------
Male
Female
```

has relatively few distinct values.

This column:

```text
email
-----
ali1@example.com
ali2@example.com
...
```

has many distinct values.

An index is often more useful when a condition can narrow the search to a relatively small portion of the table.

But again, the optimizer decides based on the specific query and data distribution.

---

# Practice

### Exercise 1 — Easy

Create an index on `email`.

```sql id="v7n2mx"
-- Your query here
```

**Answer:**

```sql id="g3q8wa"
CREATE INDEX idx_users_email
ON users(email);
```

### Exercise 2 — Medium

Create an index on `orders.user_id`.

```sql id="n8m4tc"
-- Your query here
```

**Answer:**

```sql id="z5q1rx"
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

### Exercise 3 — Hard

You frequently run:

```sql id="2p7k4d"
SELECT *
FROM orders
WHERE customer_id = 10
  AND status = 'completed';
```

Create an appropriate composite index.

**Answer:**

```sql id="c9v3ht"
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

---

# FAQ

**Q: Does an index store another copy of the table?**

Not usually as a complete copy of the table. It stores an additional data structure containing indexed values and references to the corresponding rows.

**Q: Should every column have an index?**

No.

Indexes have costs, so create them based on actual query and workload requirements.

**Q: Are primary keys automatically indexed?**

In most relational database systems, yes. The exact implementation is DBMS-specific.

**Q: Are foreign keys automatically indexed?**

Not necessarily.

For example, you may need to explicitly create:

```sql id="s2m7pq"
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

**Q: Can one table have multiple indexes?**

Yes.

For example:

```text id="k6n2vy"
users
 ├── primary key index
 ├── email index
 ├── country index
 └── created_at index
```

But too many indexes can hurt write performance.

**Q: What's a composite index?**

An index containing multiple columns:

```sql id="w5p1az"
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

**Q: Can I index a column used in JOIN?**

Yes. Join columns are common candidates for indexing, especially foreign-key columns on large tables.

**Q: How do I know whether an index is actually being used?**

Use an execution plan:

```sql id="f8c2yd"
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 10;
```

The exact `EXPLAIN` output differs between PostgreSQL, MySQL, SQL Server, and other databases.

---

# Key Takeaways

* 🔹 An **index helps the database find data faster**.
* 🔹 Indexes are especially useful for frequent `WHERE` and `JOIN` conditions.
* 🔹 Primary keys are typically indexed automatically.
* 🔹 Foreign-key columns may need explicit indexes.
* 🔹 Composite indexes contain multiple columns, and **column order matters**.
* 🔹 Indexes improve reads but add storage and write overhead.
* 🔹 Use `EXPLAIN` to understand whether your index is actually helping.

---

# Next Steps

For advanced SQL, the natural progression is:

`Indexes` → speed up data access
`EXPLAIN / Query Plans` → see how SQL executes your query
`Composite Indexes` → optimize multi-column searches
`Transactions` → safely manage multiple changes
`ACID` → understand transaction guarantees
`Normalization` → design efficient relational schemas
