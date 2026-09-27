# DROP TABLE

## Quick Definition

`DROP TABLE` is a **DDL command** used to completely remove a table from a database.

It removes the **table structure and all data inside it**.

### The Basic Idea

Suppose you have:

```text
users
-------------------------
id | name | email
1  | Ali  | ali@test.com
2  | Sara | sara@test.com
```

If you run:

```sql
DROP TABLE users;
```

The entire `users` table is gone.

---

## Syntax

```sql
DROP TABLE table_name;
```

Example:

```sql
DROP TABLE users;
```

This deletes the `users` table.

---

## Examples

### Beginner — Drop a table

```sql
DROP TABLE products;
```

The `products` table and all its data are removed.

---

### Intermediate — Drop only if it exists

```sql
DROP TABLE IF EXISTS products;
```

This prevents an error if the table doesn't exist.

---

### Advanced — Dropping related tables

Suppose:

```text
users
  ↓
orders
```

`orders` may have a foreign key referencing `users`.

Depending on your database, you may need to remove the dependent relationship first or use database-specific options.

For example, PostgreSQL supports:

```sql
DROP TABLE users CASCADE;
```

`CASCADE` can also remove dependent database objects, so use it carefully.

---

## Common Mistakes

### Mistake 1: Confusing `DROP` with `DELETE`

❌ If you only want to remove the rows:

```sql
DROP TABLE users;
```

This removes the **entire table**.

✅ To remove rows while keeping the table:

```sql
DELETE FROM users;
```

---

### Mistake 2: Confusing `DROP` with `TRUNCATE`

`DROP`:

```sql
DROP TABLE users;
```

→ Removes the **table itself**.

`TRUNCATE`:

```sql
TRUNCATE TABLE users;
```

→ Removes the **rows**, but keeps the table.

---

### Mistake 3: Dropping the wrong table

This is dangerous:

```sql
DROP TABLE users;
```

Always verify the table name before running `DROP TABLE`, especially in production.

---

## Performance Notes

`DROP TABLE` is generally fast compared with deleting rows one by one because the database removes the table structure rather than processing each row individually.

But the operation can have major consequences because the table and its data are removed.

---

## Practice

### Exercise 1 — Easy

Delete the `students` table completely.

```sql
DROP TABLE students;
```

### Exercise 2 — Medium

Delete the `orders` table only if it exists.

```sql
DROP TABLE IF EXISTS orders;
```

### Exercise 3 — Hard

You have:

```text
customers
orders
```

and `orders` references `customers`.

Think about what could happen if you try:

```sql
DROP TABLE customers;
```

The database may prevent the operation because `orders` depends on `customers`.

---

## FAQ

**Q: Is `DROP TABLE` DDL?**
A: Yes. `DROP TABLE` is a **DDL command**.

**Q: Does `DROP TABLE` delete the data?**
A: Yes. It removes both the data and table structure.

**Q: Can I use `DROP TABLE` to remove only some rows?**
A: No. Use `DELETE` with `WHERE`.

```sql
DELETE FROM users
WHERE id = 5;
```

**Q: What's the difference between `DROP` and `TRUNCATE`?**

```text
DROP      → removes table + data
TRUNCATE  → removes data, keeps table
DELETE    → removes rows
```

**Q: What happens if the table doesn't exist?**
A: You normally get an error. You can use:

```sql
DROP TABLE IF EXISTS users;
```

**Q: Can I undo `DROP TABLE`?**
A: Don't assume you can. Recovery depends on the database, transaction behavior, backups, and configuration. Treat `DROP TABLE` as a destructive operation.

---

## Key Takeaways

* 🔹 `DROP TABLE` completely removes a table.
* 🔹 It removes **both structure and data**.
* 🔹 `DROP` ≠ `DELETE` ≠ `TRUNCATE`.
* 🔹 `IF EXISTS` can prevent an error when the table doesn't exist.
* 🔹 Be extremely careful with `DROP TABLE` in production.

## Next Steps

Your basic DDL sequence is now:

```text
CREATE TABLE  → Create table
ALTER TABLE   → Modify table
DROP TABLE    → Remove table
TRUNCATE      → Remove all rows
```
