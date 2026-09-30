# DELETE
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

`DELETE` is a **DML (Data Manipulation Language)** command used to **remove existing rows from a table**.

Unlike `DROP TABLE`, `DELETE` does **not remove the table itself**.

### The Basic Idea

Suppose you have:

```text
users
--------------------------------
id | name | country | age
--------------------------------
1  | Ali  | Pakistan | 22
2  | Sara | India    | 25
3  | John | USA      | 30
```

To remove Ali:

```sql
DELETE FROM users
WHERE id = 1;
```

The table remains, but Ali's row is removed.

---

## Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

Example:

```sql
DELETE FROM users
WHERE id = 1;
```

* `DELETE FROM` → tells SQL which table to remove rows from
* `WHERE` → specifies which rows to delete
* `id = 1` → identifies the row to remove

---

## Examples

### Beginner — Delete one row

```sql
DELETE FROM users
WHERE id = 1;
```

Only the user with `id = 1` is deleted.

---

### Intermediate — Delete multiple rows

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

This removes **all users** whose country is Pakistan.

---

### Intermediate — Multiple conditions

```sql
DELETE FROM users
WHERE age < 18
  AND country = 'Pakistan';
```

Only Pakistani users under 18 are deleted.

---

### Advanced — Delete all rows

```sql
DELETE FROM users;
```

This removes **every row**, but the table remains.

Compare:

```text
DELETE FROM users;       → removes all rows
TRUNCATE TABLE users;    → removes all rows
DROP TABLE users;        → removes table + rows
```

---

## Common Mistakes

### Mistake 1 — Forgetting `WHERE`

⚠️ Dangerous:

```sql
DELETE FROM users;
```

This deletes **all rows**.

If you only want to delete one user:

```sql
DELETE FROM users
WHERE id = 1;
```

### Mistake 2 — Deleting the wrong rows

Before deleting, check your condition:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

If the result is correct, then:

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

This is a good habit when working with important data.

### Mistake 3 — Confusing `DELETE` with `DROP`

`DELETE`:

```sql
DELETE FROM users
WHERE id = 1;
```

→ Removes rows.

`DROP`:

```sql
DROP TABLE users;
```

→ Removes the entire table.

---

## Performance Notes

Deleting a large number of rows can be expensive.

For example:

```sql
DELETE FROM users
WHERE country = 'Pakistan';
```

If millions of rows match, the operation can take significant time and generate substantial database work.

For deleting **all rows**, `TRUNCATE` is often more efficient:

```sql
TRUNCATE TABLE users;
```

But `TRUNCATE` cannot target specific rows.

---

## Practice

### Exercise 1 — Easy

Delete the user with `id = 5`.

```sql
DELETE FROM users
WHERE id = 5;
```

### Exercise 2 — Medium

Delete products costing less than `10`.

```sql
DELETE FROM products
WHERE price < 10;
```

### Exercise 3 — Hard

Delete users from Pakistan who are younger than 18.

```sql
DELETE FROM users
WHERE country = 'Pakistan'
  AND age < 18;
```

---

## FAQ

**Q: Is `DELETE` DDL or DML?**
A: `DELETE` is **DML** because it modifies data.

**Q: Can I delete specific rows?**
A: Yes, use `WHERE`.

```sql
DELETE FROM users
WHERE id = 10;
```

**Q: What happens without `WHERE`?**
A: All rows in the table are deleted.

**Q: Does `DELETE` remove the table?**
A: No. The table structure remains.

**Q: Can I use multiple conditions?**
A: Yes.

```sql
DELETE FROM users
WHERE age > 60
  AND country = 'Pakistan';
```

**Q: What's the difference between `DELETE` and `TRUNCATE`?**

```text
DELETE     → removes rows, can use WHERE
TRUNCATE   → removes all rows, no WHERE
```

**Q: What's the difference between `DELETE` and `DROP`?**

```text
DELETE     → removes rows
DROP       → removes the entire table
```

---

## Key Takeaways

* 🔹 `DELETE` removes existing rows.
* 🔹 `WHERE` controls which rows are deleted.
* 🔹 Without `WHERE`, **all rows are deleted**.
* 🔹 The table structure remains.
* 🔹 `DELETE` is a **DML** command.

## Next Steps

You now have the core SQL commands:

```text
SELECT  → Read data
INSERT  → Add data
UPDATE  → Modify data
DELETE  → Remove data
```

And the basic DDL commands:

```text
CREATE TABLE  → Create table
ALTER TABLE   → Modify table structure
TRUNCATE      → Remove all rows
DROP TABLE    → Remove table
```
