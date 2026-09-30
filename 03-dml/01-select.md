# SELECT — TABLE DATA
## Table of Contents

- [Quick Definition](#quick-definition)
- [Syntax](#syntax)
- [Examples](#examples)
- [Common Mistakes](#common-mistakes)
- [Performance Notes](#performance-notes)
- [Practice](#practice)
- [FAQ](#faq)
- [Key Takeaways](#key-takeaways)
- [Next Steps](#next-steps)


## Quick Definition

`SELECT` is used to **retrieve data from a table**.

It does not change the table or its data. It simply asks the database to show you information.

### The Basic Idea

If you have:

```text
users
--------------------------------
id | name | country | age
--------------------------------
1  | Ali  | Pakistan | 22
2  | Sara | India    | 25
3  | John | USA      | 30
```

You can use `SELECT` to retrieve all or specific parts of this data.

## Syntax

```sql
SELECT column_name
FROM table_name;
```

To select all columns:

```sql
SELECT *
FROM table_name;
```

## Examples

### Beginner — Select all columns

```sql
SELECT *
FROM users;
```

Result:

```text
id | name | country  | age
---|------|----------|----
1  | Ali  | Pakistan | 22
2  | Sara | India    | 25
3  | John | USA      | 30
```

### Beginner — Select specific columns

```sql
SELECT name, age
FROM users;
```

Result:

```text
name | age
-----|----
Ali  | 22
Sara | 25
John | 30
```

### Intermediate — Use `WHERE`

```sql
SELECT name, age
FROM users
WHERE age > 22;
```

This returns only users whose age is greater than `22`.

### Intermediate — Use `ORDER BY`

```sql
SELECT name, age
FROM users
ORDER BY age DESC;
```

`DESC` means highest to lowest.

For lowest to highest:

```sql
SELECT name, age
FROM users
ORDER BY age ASC;
```

### Advanced — Combine clauses

```sql
SELECT name, age
FROM users
WHERE country = 'Pakistan'
ORDER BY age DESC;
```

The basic flow is:

```text
SELECT
  ↓
FROM
  ↓
WHERE
  ↓
ORDER BY
```

## Common Mistakes

### Mistake 1 — Wrong order

❌

```sql
FROM users
SELECT name;
```

✅

```sql
SELECT name
FROM users;
```

### Mistake 2 — Forgetting quotes for text

❌

```sql
SELECT *
FROM users
WHERE country = Pakistan;
```

✅

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

### Mistake 3 — Using `SELECT *` everywhere

`SELECT *` is useful for learning and quick checks, but in real applications, select only what you need:

```sql
SELECT name, email
FROM users;
```

## Performance Notes

For large tables, avoid unnecessary:

```sql
SELECT *
FROM users;
```

Prefer:

```sql
SELECT id, name, email
FROM users;
```

This can reduce the amount of data the database needs to return.

## Practice

### Exercise 1 — Easy

Show all products.

```sql
SELECT *
FROM products;
```

### Exercise 2 — Medium

Show product names and prices where the price is greater than `100`.

```sql
SELECT product_name, price
FROM products
WHERE price > 100;
```

### Exercise 3 — Hard

Show Pakistani users older than `20`, sorted from oldest to youngest.

```sql
SELECT name, age
FROM users
WHERE country = 'Pakistan'
  AND age > 20
ORDER BY age DESC;
```

## FAQ

**Q: What does `SELECT *` mean?**
A: Select all columns.

**Q: Can I select multiple columns?**
A: Yes.

```sql
SELECT name, email, age
FROM users;
```

**Q: Does `SELECT` change data?**
A: No. `SELECT` only retrieves data.

**Q: Can `SELECT` filter rows?**
A: Yes, with `WHERE`.

```sql
SELECT *
FROM users
WHERE age > 20;
```

**Q: Can I sort SELECT results?**
A: Yes, with `ORDER BY`.

```sql
SELECT *
FROM users
ORDER BY name ASC;
```

**Q: What is the difference between `SELECT` and `UPDATE`?**

```text
SELECT → read data
UPDATE → change data
```

## Key Takeaways

* 🔹 `SELECT` retrieves data.
* 🔹 `FROM` specifies the table.
* 🔹 `*` means all columns.
* 🔹 `WHERE` filters rows.
* 🔹 `ORDER BY` sorts results.
* 🔹 `SELECT` does not modify your data.

## Next Steps

Learn `SELECT` in this order:

```text
SELECT *
↓
SELECT specific columns
↓
WHERE
↓
AND / OR
↓
ORDER BY
↓
DISTINCT
↓
LIMIT
```
