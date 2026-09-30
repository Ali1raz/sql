# UPDATE
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

`UPDATE` is a **DML (Data Manipulation Language)** command used to **modify existing data** in a table.

Unlike `ALTER TABLE`, which changes the table structure, `UPDATE` changes the **values inside rows**.

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

You want to change Ali's age from `22` to `23`.

Use:

```sql
UPDATE users
SET age = 23
WHERE id = 1;
```

---

## Syntax

```sql
UPDATE table_name
SET column_name = new_value
WHERE condition;
```

The important parts:

* `UPDATE` → tells SQL which table to modify
* `SET` → specifies the new value
* `WHERE` → determines which rows to update

---

## Examples

### Beginner — Update one row

```sql
UPDATE users
SET age = 23
WHERE id = 1;
```

Before:

```text
id | name | age
1  | Ali  | 22
```

After:

```text
id | name | age
1  | Ali  | 23
```

---

### Intermediate — Update multiple columns

```sql
UPDATE users
SET age = 24,
    country = 'Pakistan'
WHERE id = 1;
```

You can change multiple columns in one `UPDATE`.

---

### Intermediate — Update multiple rows

```sql
UPDATE users
SET country = 'Pakistan'
WHERE country = 'India';
```

Every user whose country is currently `India` will be changed to `Pakistan`.

---

### Advanced — Update using a condition

```sql
UPDATE products
SET price = price * 1.10
WHERE category = 'Electronics';
```

This increases the price of every electronic product by **10%**.

---

## Common Mistakes

### Mistake 1: Forgetting `WHERE`

⚠️ Dangerous:

```sql
UPDATE users
SET country = 'Pakistan';
```

This updates **every row**.

If you only wanted Ali:

```sql
UPDATE users
SET country = 'Pakistan'
WHERE id = 1;
```

### Mistake 2: Updating the wrong rows

Always check your condition first.

```sql
SELECT *
FROM users
WHERE id = 1;
```

Then run:

```sql
UPDATE users
SET age = 23
WHERE id = 1;
```

### Mistake 3: Forgetting quotes for text

❌

```sql
UPDATE users
SET country = Pakistan
WHERE id = 1;
```

✅

```sql
UPDATE users
SET country = 'Pakistan'
WHERE id = 1;
```

---

## Performance Notes

`UPDATE` can become expensive when modifying a large number of rows.

For example:

```sql
UPDATE users
SET country = 'Pakistan';
```

may affect millions of rows.

For large tables:

* Use a precise `WHERE` condition.
* Make sure appropriate columns are indexed when useful.
* Consider transaction safety for important updates.
* Test the `WHERE` condition with `SELECT` first.

---

## Practice

### Exercise 1 — Easy

Change user `id = 2`'s age to `26`.

```sql
UPDATE users
SET age = 26
WHERE id = 2;
```

### Exercise 2 — Medium

Change the country of all users from `India` to `Pakistan`.

```sql
UPDATE users
SET country = 'Pakistan'
WHERE country = 'India';
```

### Exercise 3 — Hard

Increase all product prices in the `Electronics` category by 15%.

```sql
UPDATE products
SET price = price * 1.15
WHERE category = 'Electronics';
```

---

## FAQ

**Q: Is `UPDATE` DDL or DML?**
A: `UPDATE` is **DML** because it modifies existing data.

**Q: Can I update multiple columns?**
A: Yes.

```sql
UPDATE users
SET name = 'Ali Khan',
    age = 25
WHERE id = 1;
```

**Q: Can I update multiple rows?**
A: Yes. Every row matching the `WHERE` condition can be updated.

**Q: What happens if I don't use `WHERE`?**
A: The `UPDATE` affects **all rows** in the table.

**Q: Can I use calculations in `UPDATE`?**
A: Yes.

```sql
UPDATE products
SET price = price * 1.10;
```

**Q: Does `UPDATE` change the table structure?**
A: No. It changes the **data**, not the columns or table design.

---

## Key Takeaways

* 🔹 `UPDATE` modifies existing data.
* 🔹 `SET` specifies the new value.
* 🔹 `WHERE` specifies which rows to change.
* 🔹 Without `WHERE`, **all rows can be updated**.
* 🔹 `UPDATE` is **DML**, not DDL.

## Next Steps

Your basic DML commands:

```text
INSERT  → Add new rows
UPDATE  → Modify existing rows
DELETE  → Remove rows
SELECT  → Retrieve rows
```
