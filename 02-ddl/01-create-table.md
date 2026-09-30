# CREATE TABLE
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

`CREATE TABLE` is a **DDL (Data Definition Language)** command used to create a new table in a database.

It defines the table’s **columns, data types, and rules (constraints)**.

### The Basic Idea

Think of `CREATE TABLE` as designing an empty Excel sheet before entering data.

You decide:

```text
Table: users

id       → number
name     → text
email    → text
age      → number
```

After creating the table, you can use `INSERT` to add records.

---

## Syntax

```sql
CREATE TABLE table_name (
    column1 data_type,
    column2 data_type,
    column3 data_type
);
```

Example:

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    age INT
);
```

This creates a `users` table with three columns.

### Breaking it down

* `CREATE TABLE` → tells SQL to create a table
* `users` → table name
* `id` → column name
* `INT` → integer data type
* `VARCHAR(100)` → text with a maximum length of 100 characters
* `()` → contains the column definitions
* `;` → ends the statement

---

## Examples

### Beginner — Simple table

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    age INT
);
```

---

### Intermediate — Add constraints

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    age INT
);
```

Here:

* `PRIMARY KEY` → uniquely identifies each row
* `NOT NULL` → value is required
* `UNIQUE` → duplicate values aren't allowed

---

### Advanced — Realistic table

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    created_at DATE
);
```

Example data could later look like:

```text
product_id | product_name | price  | stock
-----------|--------------|--------|------
1          | Keyboard     | 50.00  | 20
2          | Mouse        | 25.00  | 35
```

---

## Common Mistakes

### Mistake 1: Forgetting the data type

❌

```sql
CREATE TABLE users (
    id,
    name
);
```

✅

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
);
```

Every column needs a data type.

### Mistake 2: Forgetting commas

❌

```sql
CREATE TABLE users (
    id INT
    name VARCHAR(100)
);
```

✅

```sql
CREATE TABLE users (
    id INT,
    name VARCHAR(100)
);
```

### Mistake 3: Creating a table that already exists

You may get an error if `users` already exists.

Some databases support:

```sql
CREATE TABLE IF NOT EXISTS users (
    id INT,
    name VARCHAR(100)
);
```

---

## Performance Notes

`CREATE TABLE` itself isn't normally a performance concern. The important part is designing the table correctly.

For example:

* Use appropriate data types.
* Define a proper `PRIMARY KEY`.
* Don't make every column unnecessarily large.
* Add constraints when they represent real business rules.

---

## Practice

### Exercise 1 — Easy

Create a `students` table with:

* `id`
* `name`
* `age`

```sql
CREATE TABLE students (
    id INT,
    name VARCHAR(100),
    age INT
);
```

### Exercise 2 — Medium

Create an `employees` table where `id` is the primary key and `name` cannot be empty.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    salary DECIMAL(10, 2)
);
```

### Exercise 3 — Hard

Create an `orders` table with an order ID, customer name, total amount, and order date.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    total_amount DECIMAL(10, 2),
    order_date DATE
);
```

---

## FAQ

**Q: Is `CREATE TABLE` DDL?**
A: Yes. It defines the structure of database objects.

**Q: Can I create multiple tables?**
A: Yes.

```sql
CREATE TABLE users (...);

CREATE TABLE orders (...);
```

**Q: What happens if I create a table that already exists?**
A: Usually, the database returns an error unless you use `IF NOT EXISTS` where supported.

**Q: Why do columns need data types?**
A: The database needs to know what kind of values each column can store.

**Q: What is a constraint?**
A: A rule applied to data, such as `PRIMARY KEY`, `NOT NULL`, or `UNIQUE`.

**Q: Is `CREATE TABLE` used to insert data?**
A: No. `CREATE TABLE` creates the structure. `INSERT` adds data.

---

## Key Takeaways

* 🔹 `CREATE TABLE` creates a new table.
* 🔹 Every column needs a **name + data type**.
* 🔹 Constraints define rules for your data.
* 🔹 `PRIMARY KEY`, `NOT NULL`, and `UNIQUE` are common constraints.
* 🔹 `CREATE TABLE` belongs to **DDL**.

## Next Steps

After `CREATE TABLE`, the natural DDL topics are:

`ALTER TABLE` → modify an existing table
`DROP TABLE` → delete a table
`TRUNCATE TABLE` → remove all rows while keeping the table structure
