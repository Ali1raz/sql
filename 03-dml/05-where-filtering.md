# WHERE — Filtering
## Table of Contents

- [Quick Definition](#quick-definition)
- [Examples](#examples)
- [Common Mistakes](#common-mistakes)
- [Performance Notes](#performance-notes)
- [Practice](#practice)
- [FAQ](#faq)
- [Key Takeaways](#key-takeaways)
- [Next Steps](#next-steps)


## Quick Definition

`WHERE` is used to **filter rows** based on a condition. Only rows that satisfy the condition are returned or affected.

Think of it as a **gatekeeper**:

> “Only let the rows matching this condition through.”

---

### The Basic Idea

Suppose we have:

| id | name | country  | age |
| -: | ---- | -------- | --: |
|  1 | Ali  | Pakistan |  22 |
|  2 | John | USA      |  30 |
|  3 | Sara | Pakistan |  25 |

If you want only users from Pakistan:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Result:

| id | name | country  | age |
| -: | ---- | -------- | --: |
|  1 | Ali  | Pakistan |  22 |
|  3 | Sara | Pakistan |  25 |

`WHERE` filters **rows**, not columns.

---

### Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

Example:

```sql
SELECT name, age
FROM users
WHERE age > 20;
```

Breaking it down:

* `SELECT` → columns you want
* `FROM` → table to read from
* `WHERE` → filtering condition
* `age > 20` → only rows where age is greater than 20

---

## Examples

### Beginner — Exact Match

Find users from Pakistan:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Find a specific user:

```sql
SELECT *
FROM users
WHERE id = 5;
```

---

### Intermediate — Multiple Conditions

Use `AND` when **both conditions must be true**:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan'
  AND age >= 18;
```

Use `OR` when **either condition can be true**:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan'
   OR country = 'India';
```

You can use parentheses to make logic clear:

```sql
SELECT *
FROM users
WHERE (country = 'Pakistan' OR country = 'India')
  AND age >= 18;
```

---

### Advanced — Different Filtering Operators

Greater than:

```sql
SELECT *
FROM users
WHERE age > 25;
```

Between two values:

```sql
SELECT *
FROM users
WHERE age BETWEEN 18 AND 30;
```

Match values from a list:

```sql
SELECT *
FROM users
WHERE country IN ('Pakistan', 'India', 'Bangladesh');
```

Pattern matching:

```sql
SELECT *
FROM users
WHERE name LIKE 'A%';
```

This finds names beginning with `A`.

Check for `NULL`:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

Do **not** use:

```sql
WHERE phone = NULL;
```

`NULL` needs `IS NULL` or `IS NOT NULL`.

---

## Common Mistakes

### Mistake #1: Forgetting quotes around text

Wrong:

```sql
SELECT *
FROM users
WHERE country = Pakistan;
```

Correct:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

Text values normally use single quotes.

### Mistake #2: Using `=` with NULL

Wrong:

```sql
SELECT *
FROM users
WHERE phone = NULL;
```

Correct:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

### Mistake #3: Confusing `AND` with `OR`

This:

```sql
WHERE age >= 18
  AND age <= 30
```

means age must be between 18 and 30.

This:

```sql
WHERE age < 18
   OR age > 30
```

means age is outside that range.

---

## Performance Notes

`WHERE` can significantly reduce the number of rows the database needs to process.

For large tables, indexes can make filtering much faster:

```sql
CREATE INDEX idx_users_country
ON users(country);
```

Then:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

can potentially use that index.

But don't create indexes blindly. Indexes improve reads but add storage and can make `INSERT`, `UPDATE`, and `DELETE` more expensive.

---

## Practice

### Exercise 1 — Easy

Find users older than 25.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT *
FROM users
WHERE age > 25;
```

### Exercise 2 — Medium

Find users from Pakistan who are at least 18.

**Hint:** Use `AND`.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT *
FROM users
WHERE country = 'Pakistan'
  AND age >= 18;
```

### Exercise 3 — Hard

Find users who are either from Pakistan or India and are at least 21.

**Hint:** Combine `OR`, `AND`, and parentheses.

```sql
-- Your query here
```

**Answer:**

```sql
SELECT *
FROM users
WHERE (country = 'Pakistan' OR country = 'India')
  AND age >= 21;
```

---

## FAQ

**Q: Can I use `WHERE` with `UPDATE`?**

Yes. In fact, you should be very careful to use it when updating specific rows.

```sql
UPDATE users
SET age = 30
WHERE id = 5;
```

Without `WHERE`, every row could be updated.

**Q: Can I use `WHERE` with `DELETE`?**

Yes:

```sql
DELETE FROM users
WHERE id = 5;
```

Without `WHERE`, all rows may be deleted.

**Q: What's the difference between `WHERE` and `HAVING`?**

`WHERE` filters individual rows **before grouping**.

`HAVING` filters groups **after `GROUP BY`**.

**Q: Can I use multiple conditions?**

Yes:

```sql
WHERE age >= 18
  AND country = 'Pakistan'
```

**Q: Does the order of `AND` and `OR` matter?**

Yes. Use parentheses when combining them:

```sql
WHERE (country = 'Pakistan' OR country = 'India')
  AND age >= 18;
```

**Q: Can I filter text partially?**

Yes, with `LIKE`:

```sql
WHERE name LIKE 'Ali%';
```

**Q: Does `WHERE` work with `NULL`?**

Yes, but use:

```sql
WHERE column_name IS NULL;
```

or:

```sql
WHERE column_name IS NOT NULL;
```

---

## Key Takeaways

* 🔹 `WHERE` **filters rows**.
* 🔹 Use comparison operators like `=`, `>`, `<`, `>=`, `<=`.
* 🔹 Use `AND`, `OR`, and `NOT` for more complex conditions.
* 🔹 Use `IN`, `BETWEEN`, and `LIKE` for common filtering patterns.
* 🔹 `NULL` requires `IS NULL` / `IS NOT NULL`.

---

## Next Steps

Learn these next:

`ORDER BY` → sort filtered results
`DISTINCT` → remove duplicate results
`GROUP BY` → group rows
`HAVING` → filter groups
`LIKE` → deeper pattern matching
