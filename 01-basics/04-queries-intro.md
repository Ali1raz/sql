# BASIC SQL QUERIES — INTRO
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

A **SQL query** is a command used to interact with data in a database.

The most common beginner query is `SELECT`, which is used to **retrieve data** from a table.

### The Basic Idea

Think of a database table like an Excel sheet:

```text
users
------------------------------------------------
id | name   | country | age
------------------------------------------------
1  | Ali    | Pakistan | 22
2  | Sara   | India    | 25
3  | John   | USA      | 30
```

A SQL query lets you ask the database questions like:

* Show me all users.
* Show me only names.
* Show me users from Pakistan.
* Sort users by age.

---

## Syntax

The basic SQL query structure is:

```sql
SELECT column_name
FROM table_name;
```

Example:

```sql
SELECT name
FROM users;
```

Result:

```text
name
------
Ali
Sara
John
```

### Breaking it down

```sql
SELECT name
FROM users;
```

* `SELECT` → what data you want
* `name` → column you want
* `FROM` → where the data comes from
* `users` → table name
* `;` → ends the query

---

## Examples

### Beginner — Select all columns

```sql
SELECT *
FROM users;
```

`*` means **all columns**.

---

### Beginner — Select specific columns

```sql
SELECT name, country
FROM users;
```

Result:

```text
name | country
-----|--------
Ali  | Pakistan
Sara | India
John | USA
```

---

### Intermediate — Filter data with `WHERE`

```sql
SELECT name, age
FROM users
WHERE age > 25;
```

This returns users whose age is greater than 25.

---

### Intermediate — Sort results

```sql
SELECT name, age
FROM users
ORDER BY age;
```

For descending order:

```sql
SELECT name, age
FROM users
ORDER BY age DESC;
```

---

### Advanced — Combine filtering and sorting

```sql
SELECT name, age
FROM users
WHERE country = 'Pakistan'
ORDER BY age DESC;
```

Meaning:

1. Get `name` and `age`
2. From `users`
3. Keep users from Pakistan
4. Sort them from oldest to youngest

---

## Common Mistakes

### Mistake 1: Wrong clause order

❌ Wrong:

```sql
FROM users
SELECT name;
```

✅ Correct:

```sql
SELECT name
FROM users;
```

The basic order is:

```text
SELECT
FROM
WHERE
ORDER BY
```

---

### Mistake 2: Forgetting quotes around text

❌ Wrong:

```sql
SELECT *
FROM users
WHERE country = Pakistan;
```

✅ Correct:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Text values normally use single quotes.

---

### Mistake 3: Using `=` with `NULL`

❌ Wrong:

```sql
SELECT *
FROM users
WHERE country = NULL;
```

✅ Correct:

```sql
SELECT *
FROM users
WHERE country IS NULL;
```

---

## Performance Notes

For learning, `SELECT *` is fine.

In real applications, prefer selecting only the columns you need:

```sql
SELECT name, email
FROM users;
```

instead of:

```sql
SELECT *
FROM users;
```

This can reduce unnecessary data being read and transferred.

---

## Practice

### Exercise 1 — Easy

Get all users.

```sql
SELECT *
FROM users;
```

### Exercise 2 — Medium

Get only the `name` and `age` of users older than 20.

```sql
SELECT name, age
FROM users
WHERE age > 20;
```

### Exercise 3 — Hard

Get Pakistani users older than 20 and sort them by age from highest to lowest.

```sql
SELECT name, age
FROM users
WHERE country = 'Pakistan'
  AND age > 20
ORDER BY age DESC;
```

---

## FAQ

**Q: What does `SELECT` do?**
A: It chooses the data you want to retrieve.

```sql
SELECT name
FROM users;
```

**Q: What does `FROM` do?**
A: It tells SQL which table to get the data from.

**Q: What does `*` mean?**
A: All columns.

```sql
SELECT *
FROM users;
```

**Q: What does `WHERE` do?**
A: Filters rows based on a condition.

```sql
WHERE age > 20
```

**Q: What does `ORDER BY` do?**
A: Sorts the result.

```sql
ORDER BY age DESC;
```

**Q: Can I use multiple conditions?**
A: Yes, with `AND` or `OR`.

```sql
WHERE age > 20
  AND country = 'Pakistan';
```

**Q: Does the order of SQL clauses matter?**
A: Yes. A basic query follows:

```text
SELECT → FROM → WHERE → ORDER BY
```

---

## Key Takeaways

* 🔹 `SELECT` → choose columns
* 🔹 `FROM` → choose table
* 🔹 `WHERE` → filter rows
* 🔹 `ORDER BY` → sort results
* 🔹 `*` → all columns

## Next Steps

After basic queries, learn these in order:

1. `WHERE` conditions
2. `AND`, `OR`, `NOT`
3. `ORDER BY`
4. `DISTINCT`
5. `LIMIT`
6. `INSERT`, `UPDATE`, `DELETE`
7. Aggregate functions like `COUNT()`, `SUM()`, `AVG()`
