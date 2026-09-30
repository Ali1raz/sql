# Execution Plan

An **execution plan** shows how the database intends to execute a SQL query. It tells you things like which indexes are used, how tables are scanned, how joins are performed, and where the query spends its time.

### The Basic Idea

Think of an execution plan as the database's **route map** for your query.

You write:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

The database decides:

```text
Query
  ↓
Find users table
  ↓
Scan table or use index?
  ↓
Filter matching rows
  ↓
Return results
```

Your job when optimizing is to inspect this route and find the expensive parts.

### Syntax

PostgreSQL:

```sql
EXPLAIN
SELECT id, name
FROM users
WHERE country = 'Pakistan';
```

For actual execution statistics:

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

### Examples

#### Beginner Example

```sql
EXPLAIN
SELECT id, name
FROM users
WHERE country = 'Pakistan';
```

You may see something conceptually like:

```text
Seq Scan on users
  Filter: country = 'Pakistan'
```

`Seq Scan` means the database is scanning the table sequentially.

If the table is small, that's perfectly reasonable.

If the table contains millions of rows, you may investigate whether an index would help:

```sql
CREATE INDEX idx_users_country
ON users(country);
```

Then check the plan again.

#### Intermediate Example

Consider:

```sql
EXPLAIN
SELECT
    o.id,
    o.total,
    c.name
FROM orders o
JOIN customers c
    ON o.customer_id = c.id
WHERE o.status = 'completed';
```

The execution plan may contain operations such as:

```text
Index Scan
Nested Loop
Hash Join
Seq Scan
Filter
```

The database is effectively answering:

```text
How do I find the orders?
How do I find their customers?
Which join strategy should I use?
Where should filtering happen?
```

You shouldn't assume that `Nested Loop` is bad or `Seq Scan` is bad. Their usefulness depends on the data and query.

#### Advanced Example

Use actual execution statistics:

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

A PostgreSQL plan might contain:

```text
Hash Join
  -> Seq Scan on orders
       Filter: status = 'completed'
  -> Hash
       -> Seq Scan on customers
  -> GroupAggregate
```

Important things to investigate include:

```text
estimated rows
actual rows
estimated cost
actual execution time
number of loops
scan type
join type
```

If the database estimates:

```text
rows=100
```

but actually processes:

```text
rows=500000
```

that mismatch can indicate inaccurate statistics or a query/data distribution problem.

### Common Mistakes

**Mistake #1: Assuming every sequential scan is bad**

```text
Seq Scan on users
```

A sequential scan can be completely reasonable for a small table or when a large percentage of rows must be returned.

The mistake is judging an operation without considering the amount of data and the overall plan.

**Mistake #2: Looking only at estimated cost**

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 10;
```

`EXPLAIN` gives estimates.

For actual runtime behavior:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 10;
```

Use the appropriate command for the database you're working with.

**Mistake #3: Adding indexes without checking the plan**

Don't immediately create five indexes because a query is slow.

First inspect:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 10;
```

Then determine what operation is actually expensive.

**Mistake #4: Ignoring estimated vs actual rows**

If the optimizer expects:

```text
rows=10
```

but gets:

```text
actual rows=100000
```

the optimizer may have made a poor decision because its assumptions about the data were inaccurate.

### Performance Notes

The main execution-plan concepts to recognize are:

| Operation         | Meaning                                                 |
| ----------------- | ------------------------------------------------------- |
| `Seq Scan`        | Reads a table sequentially                              |
| `Index Scan`      | Uses an index to locate rows                            |
| `Index Only Scan` | Gets required data directly from an index when possible |
| `Nested Loop`     | Repeatedly searches one side for each row from another  |
| `Hash Join`       | Builds a hash structure to join datasets                |
| `Merge Join`      | Joins sorted inputs                                     |
| `Sort`            | Sorts rows                                              |
| `Aggregate`       | Performs operations such as `SUM()` or `COUNT()`        |
| `Filter`          | Removes rows that don't satisfy a condition             |

For performance, pay particular attention to:

```text
Actual Time
Actual Rows
Loops
Large scans
Expensive sorts
Join operations
Rows removed by filters
Estimated vs actual rows
```

A useful mental model:

```text
Query
  ↓
Execution Plan
  ↓
Scan → Filter → Join → Sort → Aggregate
  ↓
Result
```

The goal isn't to make every operation disappear. The goal is to make the **overall work appropriate for the amount of data**.

### Practice

**Exercise 1 — Easy**

Generate an execution plan:

```sql
EXPLAIN
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Identify:

* Scan type
* Estimated rows
* Estimated cost

**Exercise 2 — Medium**

Create an index:

```sql
CREATE INDEX idx_users_country
ON users(country);
```

Then compare:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Look for changes in the scan method and execution time.

**Exercise 3 — Hard**

Analyze this:

```sql
EXPLAIN ANALYZE
SELECT
    c.name,
    COUNT(o.id) AS order_count,
    SUM(o.total) AS total_spent
FROM customers c
LEFT JOIN orders o
    ON o.customer_id = c.id
WHERE c.country = 'Pakistan'
GROUP BY c.name;
```

Find:

1. How customers are scanned.
2. How orders are accessed.
3. Which join strategy is used.
4. How many rows each operation processes.
5. Which operation consumes the most time.

### FAQ

**Q: Is an execution plan the same as the SQL query?**

A: No. The SQL describes **what** you want. The execution plan describes **how** the database intends to get it.

**Q: What is the difference between `EXPLAIN` and `EXPLAIN ANALYZE`?**

A: `EXPLAIN` shows the planned execution. `EXPLAIN ANALYZE` executes the query and reports actual execution information.

**Q: Is `Seq Scan` always bad?**

A: No. For small tables or queries returning many rows, sequential scanning can be the fastest option.

**Q: What does `cost` mean?**

A: It is the optimizer's estimate of the relative work required. It isn't simply the query's runtime in milliseconds.

**Q: What does `actual rows` tell me?**

A: It tells you how many rows an operation actually produced.

**Q: Why compare estimated rows with actual rows?**

A: Large differences can indicate inaccurate statistics or assumptions about the data, potentially leading to poor execution decisions.

**Q: Which join is fastest?**

A: There isn't one universally fastest join. `Nested Loop`, `Hash Join`, and `Merge Join` are useful under different conditions.

**Q: Can I optimize a query just by looking at the execution plan?**

A: The plan is the starting point. Combine it with query requirements, data size, indexes, and actual runtime measurements.

### Key Takeaways

* 🔹 An **execution plan is the database's roadmap** for executing a query.
* 🔹 `EXPLAIN` shows the planned operations.
* 🔹 `EXPLAIN ANALYZE` shows actual execution behavior.
* 🔹 Learn to recognize scans, joins, sorts, and aggregations.
* 🔹 Compare **estimated rows vs actual rows** when diagnosing problems.

### Next Steps

Prerequisites: `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, **Indexes**, **Query Optimization**

Next concepts: **Index Scans → Composite Indexes → Join Optimization → Query Profiling → Database Performance Tuning**
