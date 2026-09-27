# TRUNCATE TABLE

## Quick Definition

`TRUNCATE TABLE` is a **DDL command** used to remove **all rows** from a table while keeping the table structure.

### The Basic Idea

Suppose you have:

```text
users
-------------------------
id | name | country
1  | Ali  | Pakistan
2  | Sara | India
3  | John | USA
```

Run:

```sql
TRUNCATE TABLE users;
```

Now:

```text
users
-------------------------
id | name | country
```

The table still exists, but **all rows are gone**.

---

## Syntax

```sql
TRUNCATE TABLE table_name;
```

Example:

```sql
TRUNCATE TABLE users;
```

That's it. There is normally **no `WHERE` clause**.

---

## Examples

### Beginner — Remove all data

```sql
TRUNCATE TABLE users;
```

Result:

```text
users table → still exists
rows        → removed
columns     → remain
```

---

### Intermediate — Compare with `DELETE`

Using `DELETE`:

```sql
DELETE FROM users;
```

Using `TRUNCATE`:

```sql
TRUNCATE TABLE users;
```

Both can remove all rows, but they work differently internally and have different database-specific behavior.

If you want to remove **specific rows**, use `DELETE`:

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

You cannot do this with `TRUNCATE`.

---

### Advanced — Identity/auto-increment behavior

In some database systems, `TRUNCATE` can reset an identity/auto-increment counter; behavior varies by database.

For example, after:

```sql
TRUNCATE TABLE users;
```

the next inserted ID may start again from the initial value, depending on the database system and configuration.

Don't assume this behavior is identical across MySQL, PostgreSQL, SQL Server, and Oracle.

---

## Common Mistakes

### Mistake 1: Trying to use `WHERE`

❌

```sql
TRUNCATE TABLE users
WHERE country = 'Pakistan';
```

`TRUNCATE` doesn't support row filtering.

✅ Use `DELETE`:

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

---

### Mistake 2: Thinking `TRUNCATE` removes the table

❌

```sql
TRUNCATE TABLE users;
```

This does **not** remove the table.

The table structure remains.

To remove the table completely:

```sql
DROP TABLE users;
```

---

### Mistake 3: Using it without checking the table

```sql
TRUNCATE TABLE users;
```

This removes **every row** from `users`.

There is no `WHERE` to save you from accidentally targeting all rows.

---

## Performance Notes

`TRUNCATE` is generally much faster than deleting millions of rows one by one with:

```sql
DELETE FROM users;
```

It's useful when you want to **empty an entire table quickly**.

However, exact behavior regarding logging, locking, identity values, foreign keys, and rollback depends on the database system.

---

## Practice

### Exercise 1 — Easy

Remove all rows from `students` while keeping the table.

```sql
TRUNCATE TABLE students;
```

### Exercise 2 — Medium

You only want to remove users from Pakistan.

Don't use `TRUNCATE`.

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

### Exercise 3 — Hard

You want to completely remove the `users` table.

Use:

```sql
DROP TABLE users;
```

---

## FAQ

**Q: Is `TRUNCATE` DDL?**
A: Yes, it is generally classified as **DDL**.

**Q: Does `TRUNCATE` delete the table?**
A: No. It removes the rows but keeps the table structure.

**Q: Can I use `WHERE` with `TRUNCATE`?**
A: No.

**Q: How do I remove only selected rows?**
A: Use `DELETE`.

```sql
DELETE FROM users
WHERE age < 18;
```

**Q: What's the difference between `TRUNCATE` and `DROP`?**

```text
TRUNCATE → removes all rows, keeps table
DROP      → removes table + rows
```

**Q: What's the difference between `TRUNCATE` and `DELETE`?**

```text
TRUNCATE → removes all rows
DELETE   → can remove selected rows
```

**Q: Is `TRUNCATE` faster than `DELETE`?**
A: Generally yes when removing all rows, because it is designed for quickly emptying a table. Exact performance depends on the database.

---

## Key Takeaways

* 🔹 `TRUNCATE TABLE` removes **all rows**.
* 🔹 The **table remains**.
* 🔹 You cannot use `WHERE` with `TRUNCATE`.
* 🔹 Use `DELETE` when you need to remove specific rows.
* 🔹 Use `DROP` when you want to remove the entire table.

## Next Steps

Your DDL basics are now:

```text
CREATE TABLE  → Create structure
ALTER TABLE   → Modify structure
TRUNCATE      → Remove all rows
DROP TABLE    → Remove structure + rows
```
