# ALTER TABLE

## Quick Definition

`ALTER TABLE` is a **DDL command** used to modify the structure of an existing table.

You can use it to **add, modify, rename, or remove columns**, and in many databases, to change constraints.

### The Basic Idea

Think of `ALTER TABLE` as editing the design of an existing table.

You already have:

```text
users
-------------------------
id | name | age
```

Later, you decide you need an `email` column:

```text
users
--------------------------------
id | name | age | email
```

You don't need to recreate the table. You use `ALTER TABLE`.

---

## Syntax

Basic structure:

```sql
ALTER TABLE table_name
action;
```

For example:

```sql
ALTER TABLE users
ADD email VARCHAR(150);
```

This adds an `email` column to the existing `users` table.

---

## Examples

### Beginner — Add a column

```sql
ALTER TABLE users
ADD email VARCHAR(150);
```

Before:

```text
id | name | age
```

After:

```text
id | name | age | email
```

---

### Intermediate — Add multiple columns

Some databases support:

```sql
ALTER TABLE users
ADD phone VARCHAR(20),
ADD city VARCHAR(100);
```

Now the table has:

```text
id | name | age | email | phone | city
```

---

### Intermediate — Rename a column

Syntax differs between database systems, but a common form is:

```sql
ALTER TABLE users
RENAME COLUMN name TO full_name;
```

Now:

```text
id | full_name | age | email
```

---

### Intermediate — Drop a column

```sql
ALTER TABLE users
DROP COLUMN age;
```

The `age` column is removed.

⚠️ The column and its stored data are deleted, so be careful with this operation.

---

### Advanced — Add a constraint

For example, adding a unique constraint:

```sql
ALTER TABLE users
ADD CONSTRAINT unique_user_email
UNIQUE (email);
```

Now duplicate emails are not allowed.

---

## Common Mistakes

### Mistake 1: Forgetting `ALTER TABLE`

❌

```sql
ADD email VARCHAR(150);
```

✅

```sql
ALTER TABLE users
ADD email VARCHAR(150);
```

---

### Mistake 2: Dropping a column accidentally

❌

```sql
ALTER TABLE users
DROP COLUMN email;
```

This permanently removes the column and its data.

Always verify the column before running `DROP COLUMN`.

---

### Mistake 3: Assuming syntax is identical everywhere

`ALTER TABLE` syntax can vary between **MySQL, PostgreSQL, SQL Server, Oracle**, etc.

For example, changing a column's data type has different syntax across databases.

---

## Performance Notes

`ALTER TABLE` can be expensive on large production tables.

For example:

```sql
ALTER TABLE users
ADD email VARCHAR(150);
```

may be quick in some databases, while changing a large column or adding certain constraints may require significant work.

For large production tables, consider:

* Table size
* Existing indexes
* Existing data
* Locks/downtime
* Database-specific behavior

---

## Practice

### Exercise 1 — Easy

Add a `phone` column to `users`.

```sql
ALTER TABLE users
ADD phone VARCHAR(20);
```

### Exercise 2 — Medium

Rename `phone` to `phone_number`.

```sql
ALTER TABLE users
RENAME COLUMN phone TO phone_number;
```

### Exercise 3 — Hard

Add a unique constraint to `email`.

```sql
ALTER TABLE users
ADD CONSTRAINT unique_user_email
UNIQUE (email);
```

---

## FAQ

**Q: Does `ALTER TABLE` modify the data or structure?**
A: Mainly the **structure** of the table.

**Q: Can I add a column?**
A: Yes.

```sql
ALTER TABLE users
ADD email VARCHAR(150);
```

**Q: Can I delete a column?**
A: Yes, using `DROP COLUMN`.

```sql
ALTER TABLE users
DROP COLUMN email;
```

**Q: Can I rename a column?**
A: Yes.

```sql
ALTER TABLE users
RENAME COLUMN name TO full_name;
```

**Q: Can I rename a table?**
A: Yes, but the syntax depends on the database system. For example, PostgreSQL/MySQL commonly use:

```sql
ALTER TABLE users
RENAME TO customers;
```

**Q: Is `ALTER TABLE` DDL?**
A: Yes. `ALTER TABLE` is a **DDL command**.

---

## Key Takeaways

* 🔹 `ALTER TABLE` modifies an existing table.
* 🔹 `ADD` → add columns.
* 🔹 `DROP COLUMN` → remove columns.
* 🔹 `RENAME COLUMN` → rename columns.
* 🔹 Syntax can differ between database systems.

## Next Steps

The natural DDL sequence is:

`CREATE TABLE` → create table
`ALTER TABLE` → modify table
`DROP TABLE` → delete table
`TRUNCATE TABLE` → remove all rows, keep table structure
