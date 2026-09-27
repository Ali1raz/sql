# INSERT

## Quick Definition

`INSERT` is a **DML (Data Manipulation Language)** command used to **add new rows of data** into a table.

Think of it as putting a new record into an existing table.

### The Basic Idea

Suppose you have:

```text
users
--------------------------------
id | name | country | age
--------------------------------
1  | Ali  | Pakistan | 22
2  | Sara | India    | 25
```

You want to add John:

```sql
INSERT INTO users (id, name, country, age)
VALUES (3, 'John', 'USA', 30);
```

Now the table contains 3 rows.

## Syntax

```sql
INSERT INTO table_name (column1, column2, column3)
VALUES (value1, value2, value3);
```

Example:

```sql
INSERT INTO users (id, name, country, age)
VALUES (3, 'John', 'USA', 30);
```

The column order and value order must match:

```text
id       → 3
name     → 'John'
country  → 'USA'
age      → 30
```

## Examples

### Beginner — Insert one row

```sql
INSERT INTO users (id, name, country, age)
VALUES (3, 'John', 'USA', 30);
```

### Intermediate — Insert multiple rows

```sql
INSERT INTO users (id, name, country, age)
VALUES
    (4, 'Ahmed', 'Pakistan', 24),
    (5, 'Emma', 'UK', 27),
    (6, 'David', 'Canada', 31);
```

This adds three rows in one statement.

### Intermediate — Insert only selected columns

If some columns have defaults or allow `NULL`:

```sql
INSERT INTO users (name, country)
VALUES ('Ali', 'Pakistan');
```

The other columns will use their default values or `NULL`, depending on the table definition.

### Advanced — Insert using `SELECT`

You can insert the result of another query:

```sql
INSERT INTO customers (name, country)
SELECT name, country
FROM users
WHERE country = 'Pakistan';
```

This copies matching data from `users` into `customers`.

## Common Mistakes

### Mistake 1 — Number of columns doesn't match values

❌

```sql
INSERT INTO users (id, name, age)
VALUES (3, 'John');
```

There are 3 columns but only 2 values.

✅

```sql
INSERT INTO users (id, name, age)
VALUES (3, 'John', 30);
```

### Mistake 2 — Forgetting quotes around text

❌

```sql
INSERT INTO users (name, country)
VALUES (John, Pakistan);
```

✅

```sql
INSERT INTO users (name, country)
VALUES ('John', 'Pakistan');
```

### Mistake 3 — Duplicate primary key

If `id` is a primary key:

```sql
INSERT INTO users (id, name)
VALUES (1, 'David');
```

If `id = 1` already exists, the database will normally reject the insert.

### Mistake 4 — Inserting `NULL` into a `NOT NULL` column

If:

```sql
name VARCHAR(100) NOT NULL
```

then this can fail:

```sql
INSERT INTO users (id)
VALUES (10);
```

because `name` is required.

## Performance Notes

For inserting many rows, a multi-row `INSERT` is generally more efficient than running many separate statements:

```sql
INSERT INTO users (id, name)
VALUES
    (1, 'Ali'),
    (2, 'Sara'),
    (3, 'John');
```

For very large imports, databases often provide specialized bulk-loading tools.

## Practice

### Exercise 1 — Easy

Insert a user:

```text
id = 7
name = Ahmed
country = Pakistan
age = 23
```

Solution:

```sql
INSERT INTO users (id, name, country, age)
VALUES (7, 'Ahmed', 'Pakistan', 23);
```

### Exercise 2 — Medium

Insert two products:

```sql
INSERT INTO products (product_id, product_name, price)
VALUES
    (1, 'Keyboard', 50.00),
    (2, 'Mouse', 25.00);
```

### Exercise 3 — Hard

Insert Pakistani users into another table:

```sql
INSERT INTO customers (name, country)
SELECT name, country
FROM users
WHERE country = 'Pakistan';
```

## FAQ

**Q: Is `INSERT` DDL or DML?**
A: `INSERT` is **DML** because it modifies table data.

**Q: Can I insert multiple rows at once?**
A: Yes.

```sql
INSERT INTO users (id, name)
VALUES
    (1, 'Ali'),
    (2, 'Sara');
```

**Q: Do I have to specify every column?**
A: No. You can specify only the columns you want to provide, as long as the remaining columns allow `NULL` or have default values.

**Q: What happens if I don't specify the column names?**
A: You can write:

```sql
INSERT INTO users
VALUES (3, 'John', 'USA', 30);
```

But this depends on the exact column order and requires values for the expected columns. Specifying column names is safer.

**Q: Can I insert `NULL`?**
A: Yes, if the column allows `NULL`.

```sql
INSERT INTO users (id, name, age)
VALUES (8, 'David', NULL);
```

**Q: What's the difference between `INSERT` and `UPDATE`?**

```text
INSERT → adds a new row
UPDATE → changes an existing row
```

## Key Takeaways

* 🔹 `INSERT` adds new rows.
* 🔹 `INTO` specifies the table.
* 🔹 `VALUES` provides the data.
* 🔹 Column order must match value order.
* 🔹 `INSERT` is a **DML** command.

## Next Steps

Your basic DML commands are:

```text
SELECT  → Read data
INSERT  → Add data
UPDATE  → Change data
DELETE  → Remove data
```
