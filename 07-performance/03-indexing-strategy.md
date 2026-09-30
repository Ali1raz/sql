# INDEXING STRATEGIES
## Table of Contents

- [Quick Definition](#quick-definition)
- [Syntax](#syntax)
- [Examples](#examples)
- [Common Indexing Strategies](#common-indexing-strategies)
- [Common Mistakes](#common-mistakes)
- [Performance Notes](#performance-notes)
- [Practice](#practice)
- [FAQ](#faq)
- [Key Takeaways](#key-takeaways)
- [Next Steps](#next-steps)


## Quick Definition

**Indexing strategies** are methods for deciding **which columns to index, what type of index to use, and how to structure indexes** so queries run efficiently without creating unnecessary overhead.

### The Basic Idea

Think of an index like the index at the back of a book.

Without an index:

```text
Database
   ↓
Check row 1
Check row 2
Check row 3
...
Check row 1,000,000
```

With a useful index:

```text
Index
  ↓
Find matching values
  ↓
Locate required rows
  ↓
Return results
```

But indexes aren't free. They consume storage and can slow down `INSERT`, `UPDATE`, and `DELETE`.

---

## Syntax

Basic index:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Unique index:

```sql
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

Composite index:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

Remove an index:

```sql
DROP INDEX idx_users_email;
```

PostgreSQL:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

MySQL:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

### Breaking It Down

```sql
CREATE INDEX idx_users_email
ON users(email);
```

* `CREATE INDEX` → creates an index
* `idx_users_email` → index name
* `ON users` → table being indexed
* `(email)` → indexed column

---

## Examples

### Beginner Example

Suppose this query runs frequently:

```sql
SELECT id, name, email
FROM users
WHERE email = 'tony@example.com';
```

Create an index:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now the database has an additional structure for locating users by email.

For a column that must be unique:

```sql
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

This both supports lookup and prevents duplicate emails.

---

### Intermediate Example — Composite Index

Consider:

```sql
SELECT id, total
FROM orders
WHERE customer_id = 101
  AND status = 'completed';
```

A composite index can support this access pattern:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

The **column order matters**.

```text
(customer_id, status)
```

is different from:

```text
(status, customer_id)
```

The correct order depends on the queries your application actually runs.

For example:

```sql
WHERE customer_id = 101
```

can generally benefit from:

```text
(customer_id, status)
```

because `customer_id` is the leading column.

---

### Advanced Example — Partial Index

Suppose an orders table contains millions of rows, but your application frequently queries only completed orders.

PostgreSQL supports:

```sql
CREATE INDEX idx_completed_orders
ON orders(customer_id)
WHERE status = 'completed';
```

Then:

```sql
SELECT id, total
FROM orders
WHERE customer_id = 101
  AND status = 'completed';
```

The index contains only rows satisfying the condition.

This can reduce index size and maintenance compared with indexing every row.

---

## Common Indexing Strategies

### 1. Index frequently searched columns

Good candidates often include:

```sql
WHERE email = ?
WHERE customer_id = ?
WHERE order_id = ?
```

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

---

### 2. Index foreign keys when appropriate

If you frequently join:

```sql
SELECT *
FROM customers c
JOIN orders o
    ON o.customer_id = c.id;
```

An index on the referencing column can help:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

---

### 3. Use composite indexes for multi-column queries

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'completed';
```

Possible index:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

---

### 4. Consider sorting requirements

If this query is frequent:

```sql
SELECT id, name, created_at
FROM users
ORDER BY created_at DESC
LIMIT 20;
```

An index may help:

```sql
CREATE INDEX idx_users_created_at
ON users(created_at DESC);
```

---

### 5. Don't index everything

This is usually a poor strategy:

```sql
CREATE INDEX idx_users_name ON users(name);
CREATE INDEX idx_users_age ON users(age);
CREATE INDEX idx_users_city ON users(city);
CREATE INDEX idx_users_country ON users(country);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_created_at ON users(created_at);
```

Every index has a cost.

The question isn't:

> "Can I index this column?"

It's:

> "Does this index support an important workload enough to justify its cost?"

---

## Common Mistakes

### Mistake #1 — Indexing low-value columns blindly

Example:

```sql
CREATE INDEX idx_users_gender
ON users(gender);
```

If a table has only two possible values and most queries return a large portion of the table, this index may provide little benefit.

**Fix:** Check actual query patterns and execution plans.

---

### Mistake #2 — Wrong composite index order

Suppose:

```sql
CREATE INDEX idx_orders_status_customer
ON orders(status, customer_id);
```

But your important query is primarily:

```sql
WHERE customer_id = 101
```

The index may not be as useful as:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

The **leading column matters**.

---

### Mistake #3 — Too many indexes

Imagine:

```text
users
 ├── idx_email
 ├── idx_name
 ├── idx_city
 ├── idx_country
 ├── idx_phone
 ├── idx_created_at
 └── idx_status
```

Every write may need to maintain these indexes.

**Fix:** Remove indexes that don't provide meaningful workload benefits.

---

### Mistake #4 — Function prevents normal index usage

Potentially problematic:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'tony@example.com';
```

A normal index on `email` may not support this expression efficiently.

Depending on the database, an expression/function-based index can be appropriate.

PostgreSQL example:

```sql
CREATE INDEX idx_users_lower_email
ON users(LOWER(email));
```

---

### Mistake #5 — Assuming an index guarantees improvement

Creating:

```sql
CREATE INDEX idx_orders_status
ON orders(status);
```

doesn't guarantee that this:

```sql
SELECT *
FROM orders
WHERE status = 'completed';
```

will become faster.

The optimizer may decide that scanning the table is cheaper.

Check:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE status = 'completed';
```

---

## Performance Notes

Indexes generally improve **read performance** but add overhead to **write operations**.

```text
SELECT
  ↑
Indexes can help

INSERT
UPDATE
DELETE
  ↑
Indexes must be maintained
```

Consider these factors:

| Factor          | Why it matters                                         |
| --------------- | ------------------------------------------------------ |
| Table size      | Large tables often benefit more from selective indexes |
| Query frequency | Frequently executed queries justify optimization       |
| Selectivity     | More selective values can make indexes more useful     |
| Column order    | Critical for composite indexes                         |
| Write frequency | Many indexes can increase write cost                   |
| Storage         | Indexes consume disk/memory                            |
| Query pattern   | Indexes should match actual workload                   |

A useful workflow:

```text
Slow query
    ↓
EXPLAIN ANALYZE
    ↓
Identify expensive operation
    ↓
Check existing indexes
    ↓
Create/modify index
    ↓
Run EXPLAIN ANALYZE again
    ↓
Compare results
```

Don't guess. **Measure before and after.**

---

## Practice

### Exercise 1 — Easy

Given:

```sql
SELECT *
FROM users
WHERE email = 'tony@example.com';
```

Create an index.

**Solution:**

```sql
CREATE INDEX idx_users_email
ON users(email);
```

---

### Exercise 2 — Medium

Given:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'completed';
```

Create a composite index.

**Solution:**

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

Then inspect it:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 101
  AND status = 'completed';
```

---

### Exercise 3 — Hard

Your application frequently runs:

```sql
SELECT id, total
FROM orders
WHERE customer_id = 101
  AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;
```

Think about whether this index could support the query:

```sql
CREATE INDEX idx_orders_customer_status_created
ON orders(customer_id, status, created_at DESC);
```

Then test it with:

```sql
EXPLAIN ANALYZE
SELECT id, total
FROM orders
WHERE customer_id = 101
  AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;
```

Compare the plan before and after the index.

---

## FAQ

**Q: When should I create an index?**

A: When an important query repeatedly filters, joins, sorts, or looks up data using that column and the performance benefit justifies the index's cost.

**Q: What's the difference between a single-column and composite index?**

A: A single-column index covers one column:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

A composite index covers multiple columns:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

**Q: Does column order matter in a composite index?**

A: Yes. `(customer_id, status)` and `(status, customer_id)` are not interchangeable. The leading column is particularly important.

**Q: Should every foreign key have an index?**

A: Not automatically. Indexing frequently joined or filtered foreign-key columns is often useful, but inspect the workload and execution plans.

**Q: Can indexes slow down INSERT and UPDATE?**

A: Yes. The database must maintain affected indexes when data changes.

**Q: Why didn't my database use the index?**

A: The optimizer may have determined that another access method is cheaper. Table size, selectivity, statistics, and the query itself all matter.

**Q: Can I have too many indexes?**

A: Yes. Excessive indexes consume storage and can increase write and maintenance costs.

**Q: How do I know whether an index actually helped?**

A: Compare execution plans and measured runtime before and after:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 101;
```

---

## Key Takeaways

* 🔹 **Indexes are built for query patterns, not simply for columns.**
* 🔹 Composite index **column order matters**.
* 🔹 Indexes can speed up reads but increase write and storage costs.
* 🔹 Don't create indexes blindly; use `EXPLAIN ANALYZE`.
* 🔹 The best index is the one that improves an important workload enough to justify its cost.

## Next Steps

**Prerequisites:** `WHERE`, `JOIN`, `ORDER BY`, `EXPLAIN`, `Execution Plans`

**Next:** `Composite Indexes` → `Covering Indexes` → `Index Scans` → `Query Optimization` → `Database Performance Tuning`
