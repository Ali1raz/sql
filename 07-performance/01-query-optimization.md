# Query Optimization

Query optimization is the process of making SQL queries run **faster and use fewer database resources** while still returning the correct result. It involves improving the query itself, indexes, joins, filtering, and how the database executes the query.

### The Basic Idea

Think of a database like a warehouse. A bad query makes the worker search every box in the warehouse. An optimized query gives the worker a precise location and asks them to inspect only what matters.

The main goals are:

* Reduce unnecessary rows
* Reduce unnecessary columns
* Use appropriate indexes
* Avoid expensive operations when possible
* Make joins efficient
* Check the execution plan

### Syntax

There is no single `OPTIMIZE QUERY` command. Optimization usually involves changing the query and inspecting its execution plan.

PostgreSQL:

```sql
EXPLAIN ANALYZE
SELECT id, name
FROM users
WHERE country = 'Pakistan';
```

MySQL:

```sql
EXPLAIN
SELECT id, name
FROM users
WHERE country = 'Pakistan';
```

`EXPLAIN` shows how the database plans to execute the query.

`EXPLAIN ANALYZE` actually executes the query and provides runtime information.

### Examples

#### Beginner Example

Avoid retrieving unnecessary columns:

```sql
-- Less efficient
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Better:

```sql
SELECT id, name, email
FROM users
WHERE country = 'Pakistan';
```

If the table has 30 columns but you need only 3, don't make the database retrieve all 30.

#### Intermediate Example

Suppose we frequently search users by country:

```sql
SELECT id, name, email
FROM users
WHERE country = 'Pakistan';
```

An index can help:

```sql
CREATE INDEX idx_users_country
ON users(country);
```

Now the database can use the index to locate matching rows instead of potentially scanning the entire table.

Check whether it is being used:

```sql
EXPLAIN ANALYZE
SELECT id, name, email
FROM users
WHERE country = 'Pakistan';
```

#### Advanced Example

Consider:

```sql
SELECT
    c.id,
    c.name,
    SUM(o.total) AS total_spent
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'completed'
GROUP BY c.id, c.name;
```

Useful indexes might include:

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

But don't blindly create indexes. The database's execution plan should guide the decision.

For a large production database:

```sql
EXPLAIN ANALYZE
SELECT
    c.id,
    c.name,
    SUM(o.total) AS total_spent
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'completed'
GROUP BY c.id, c.name;
```

Look for expensive operations such as:

* Full/sequential table scans
* Large sorts
* Expensive joins
* Large numbers of rows being processed
* Poor index usage

### Common Mistakes

**Mistake #1: Using `SELECT *`**

```sql
SELECT *
FROM orders;
```

Better:

```sql
SELECT id, customer_id, total
FROM orders;
```

`SELECT *` can transfer and process much more data than necessary.

**Mistake #2: Applying functions to indexed columns**

Potentially problematic:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'tony@example.com';
```

An ordinary index on `email` may not be useful for this expression.

Depending on the database, consider an expression/function-based index or store/search normalized values appropriately.

**Mistake #3: Filtering too late**

Less efficient pattern:

```sql
SELECT *
FROM orders o
JOIN customers c
    ON o.customer_id = c.id
WHERE o.status = 'completed';
```

The optimizer may handle this efficiently itself, but the important lesson is not to assume that manually rewriting joins always improves performance. **Check the execution plan.**

**Mistake #4: Creating indexes everywhere**

```sql
CREATE INDEX idx1 ON users(name);
CREATE INDEX idx2 ON users(email);
CREATE INDEX idx3 ON users(country);
CREATE INDEX idx4 ON users(city);
```

More indexes aren't automatically better.

Indexes consume storage and can make `INSERT`, `UPDATE`, and `DELETE` operations more expensive.

### Performance Notes

Query performance is affected by:

* Table size
* Number of rows processed
* Indexes
* Join strategy
* Sorting and grouping
* Filtering
* Network/data transfer
* Database statistics
* Hardware and memory

A useful optimization workflow is:

```text
Slow query
   ↓
Measure it
   ↓
EXPLAIN / EXPLAIN ANALYZE
   ↓
Find expensive operation
   ↓
Change query/index/schema
   ↓
Measure again
```

Don't optimize based purely on how the SQL **looks**. The execution plan tells you what the database is actually doing.

### Practice

**Exercise 1 — Easy**

Optimize this query:

```sql
SELECT *
FROM products
WHERE category = 'Laptop';
```

Hint: Only retrieve the columns you actually need.

Solution:

```sql
SELECT id, name, price
FROM products
WHERE category = 'Laptop';
```

**Exercise 2 — Medium**

You frequently run:

```sql
SELECT id, name
FROM users
WHERE email = 'tony@example.com';
```

Create an appropriate index.

Solution:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

**Exercise 3 — Hard**

Analyze this query:

```sql
EXPLAIN ANALYZE
SELECT
    c.name,
    SUM(o.total) AS total_spent
FROM customers c
JOIN orders o
    ON o.customer_id = c.id
WHERE o.status = 'completed'
GROUP BY c.name;
```

Look at:

* How many rows are scanned
* Whether indexes are used
* Join method
* Aggregation cost
* Execution time

Then test whether an index improves the query.

### FAQ

**Q: Does an index always make a query faster?**

A: No. Indexes help specific access patterns. They also add storage and write overhead.

**Q: What is `EXPLAIN` used for?**

A: It shows the database's execution plan.

```sql
EXPLAIN
SELECT *
FROM users
WHERE country = 'Pakistan';
```

**Q: What is `EXPLAIN ANALYZE`?**

A: It executes the query and provides actual execution statistics. This is particularly useful for comparing estimates with reality.

**Q: Should I always use indexes?**

A: No. Index columns that are frequently searched, joined, sorted, or otherwise benefit from indexed access. Avoid indexing every column.

**Q: Is `SELECT *` always slow?**

A: No, but it can become expensive when tables contain many columns or large datasets. Selecting only required columns is generally better.

**Q: Why can a query ignore my index?**

A: The optimizer may determine that using the index isn't cheaper. For example, if most rows match the condition, scanning the table may be more efficient.

**Q: Can rewriting SQL improve performance?**

A: Yes, sometimes. But the optimizer may already transform equivalent queries. Use the execution plan and measurements rather than guessing.

### Key Takeaways

* 🔹 **Measure first** — don't optimize blindly.
* 🔹 Use `EXPLAIN` / `EXPLAIN ANALYZE` to understand execution.
* 🔹 Select only the columns you need.
* 🔹 Use indexes strategically.
* 🔹 Optimize expensive joins, filters, sorts, and aggregations.
* 🔹 More indexes don't automatically mean better performance.

### Next Steps

Prerequisites: `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`, and **Indexes**

Next concepts: **Execution Plans → Index Optimization → Composite Indexes → Query Profiling → Database Performance Tuning**
