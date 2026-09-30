# CONSTRAINTS

## Quick Definition

**Constraints** are rules enforced by the database to control what data can be stored in a table.

They protect your database from invalid, missing, duplicate, or inconsistent data.

Think of constraints as **security guards at the database door**: bad data gets stopped before it enters.

Common SQL constraints are:

```text
PRIMARY KEY
FOREIGN KEY
NOT NULL
UNIQUE
CHECK
DEFAULT
```

---

### The Basic Idea

Without constraints, you could accidentally insert:

```text
customer_id = NULL
email = NULL
email = same as another customer
age = -50
customer_id = 9999 when customer 9999 doesn't exist
```

Constraints prevent these problems.

For example:

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT CHECK (age >= 18),
    country VARCHAR(100) DEFAULT 'Pakistan'
);
```

The database now enforces rules for every inserted or updated row.

---

# Syntax

The general pattern is:

```sql
CREATE TABLE table_name (
    column_name data_type CONSTRAINT,
    column_name data_type CONSTRAINT
);
```

Example:

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    age INT CHECK (age >= 18),
    country VARCHAR(100) DEFAULT 'Pakistan'
);
```

You can also give constraints explicit names:

```sql
CREATE TABLE users (
    user_id INT,
    email VARCHAR(255),

    CONSTRAINT pk_users
        PRIMARY KEY (user_id),

    CONSTRAINT uq_users_email
        UNIQUE (email)
);
```

Naming constraints becomes particularly useful in larger production databases.

---

# 1. PRIMARY KEY

A `PRIMARY KEY` uniquely identifies each row.

A primary key:

* Must be unique.
* Cannot be `NULL`.
* A table normally has one primary key constraint.
* It can consist of one column or multiple columns.

Example:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

Valid:

```sql
INSERT INTO customers (customer_id, name)
VALUES (1, 'Ali');

INSERT INTO customers (customer_id, name)
VALUES (2, 'Sara');
```

Invalid:

```sql
INSERT INTO customers (customer_id, name)
VALUES (1, 'John');
```

Why?

`customer_id = 1` already exists.

---

# 2. FOREIGN KEY

A `FOREIGN KEY` ensures that a value refers to an existing row in another table.

Example:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

This prevents:

```sql
INSERT INTO orders (order_id, customer_id)
VALUES (101, 999);
```

if customer `999` doesn't exist.

The relationship is:

```text
customers
customer_id
     ↑
     |
orders
customer_id
```

---

# 3. NOT NULL

`NOT NULL` means a column **must have a value**.

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

This is invalid:

```sql
INSERT INTO users (user_id, name)
VALUES (1, NULL);
```

The database rejects it.

Use `NOT NULL` for values that your application requires.

For example:

```text
email
password_hash
created_at
product_name
order_id
```

But don't blindly make every column `NOT NULL`. Sometimes `NULL` legitimately means "unknown" or "not provided."

---

# 4. UNIQUE

`UNIQUE` prevents duplicate values.

Example:

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

This is valid:

```sql
INSERT INTO users (user_id, email)
VALUES (1, 'ali@example.com');

INSERT INTO users (user_id, email)
VALUES (2, 'sara@example.com');
```

But this is rejected:

```sql
INSERT INTO users (user_id, email)
VALUES (3, 'ali@example.com');
```

because the email already exists.

Common uses:

```text
email
username
phone_number
national_id
product_sku
```

Be careful with `NULL`: the exact behavior of multiple `NULL` values under a `UNIQUE` constraint depends on the database system and its semantics.

---

# 5. CHECK

`CHECK` ensures that a value satisfies a condition.

Example:

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2) CHECK (price >= 0)
);
```

This is invalid:

```sql
INSERT INTO products (product_id, name, price)
VALUES (1, 'Laptop', -500);
```

You can also use more complex conditions:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT CHECK (age >= 18),
    salary DECIMAL(10, 2) CHECK (salary > 0)
);
```

`CHECK` is useful for enforcing **business rules that can be expressed as row-level conditions**.

---

# 6. DEFAULT

`DEFAULT` supplies a value when one isn't provided.

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    country VARCHAR(100) DEFAULT 'Pakistan'
);
```

Now:

```sql
INSERT INTO users (user_id, name)
VALUES (1, 'Ali');
```

can produce:

```text
user_id | name | country
--------|------|---------
1       | Ali  | Pakistan
```

Another common example:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'pending'
);
```

If no status is supplied, it starts as `pending`.

---

# Combining Constraints

Real tables normally use several constraints together.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT CHECK (age >= 18),
    country VARCHAR(100) DEFAULT 'Pakistan'
);
```

Each rule has a different job:

```text
PRIMARY KEY → identifies the row
NOT NULL    → value is required
UNIQUE      → prevents duplicates
CHECK       → validates the value
DEFAULT     → supplies a value automatically
```

---

# Examples

## Beginner — Basic User Table

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    age INT CHECK (age >= 13)
);
```

Try:

```sql
INSERT INTO users (user_id, username, age)
VALUES (1, 'ali', 25);
```

Expected:

```text
user_id | username | age
--------|----------|----
1       | ali      | 25
```

This fails:

```sql
INSERT INTO users (user_id, username, age)
VALUES (2, 'ali', 25);
```

because `username` must be unique.

This also fails:

```sql
INSERT INTO users (user_id, username, age)
VALUES (3, 'john', 10);
```

because `age >= 13` is required.

---

## Intermediate — E-Commerce Schema

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price >= 0),
    stock INT DEFAULT 0 CHECK (stock >= 0)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Now the database protects several rules:

```text
Customer ID → unique
Customer name → required
Customer email → required + unique
Product price → cannot be negative
Product stock → cannot be negative
Order status → defaults to pending
Order customer → must exist
```

---

## Advanced — Named Constraints

For production schemas, explicitly naming constraints can make errors and migrations easier to understand.

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    total DECIMAL(10, 2),
    status VARCHAR(20),

    CONSTRAINT pk_orders
        PRIMARY KEY (order_id),

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id),

    CONSTRAINT chk_orders_total
        CHECK (total >= 0),

    CONSTRAINT chk_orders_status
        CHECK (status IN ('pending', 'paid', 'cancelled'))
);
```

Now the rules are explicit:

```text
pk_orders
fk_orders_customer
chk_orders_total
chk_orders_status
```

This becomes useful when diagnosing constraint violations or modifying schemas.

---

# Common Mistakes

### Mistake #1 — Using `UNIQUE` when you need `NOT NULL`

This:

```sql
email VARCHAR(255) UNIQUE
```

prevents duplicate non-null values, but it does not necessarily mean an email must be provided.

If email is required:

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

### Mistake #2 — Using `DEFAULT` as validation

This:

```sql
age INT DEFAULT 18
```

does **not** prevent:

```sql
age = -50
```

If negative ages should be rejected:

```sql
age INT DEFAULT 18 CHECK (age >= 0)
```

`DEFAULT` provides a value. `CHECK` validates a value.

### Mistake #3 — Storing relationships without a foreign key

Bad:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT
);
```

The database doesn't know whether `customer_id` actually exists.

Better:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

### Mistake #4 — Overusing `CHECK` for application logic

A database constraint is excellent for simple data integrity rules:

```sql
CHECK (price >= 0)
```

But complicated business workflows may belong in application logic, stored procedures, triggers, or transaction-level operations depending on the system.

Don't turn one simple table into a battlefield of obscure constraints.

---

# Performance Notes

Constraints aren't primarily performance features. Their main purpose is **data integrity**.

However, some constraints require indexes or index-like structures internally.

For example, `PRIMARY KEY` and `UNIQUE` constraints generally involve indexes in major relational databases.

Foreign keys can also benefit from indexes on the referencing columns, particularly when those columns are frequently used in joins or when parent rows are updated/deleted.

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

The important distinction:

```text
Constraint → "Is this data allowed?"

Index      → "Can I find this data efficiently?"
```

Don't create indexes blindly just because a column is a foreign key. Consider the actual workload and database behavior.

---

# Practice

### Exercise 1 — Easy

Create a `products` table with:

* `product_id` as primary key
* `name` required
* `price` required
* `price` cannot be negative
* `stock` defaults to `0`
* `stock` cannot be negative

**Solution:**

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock INT DEFAULT 0 CHECK (stock >= 0)
);
```

### Exercise 2 — Medium

Create a `users` table where:

* `user_id` is the primary key
* `username` is required and unique
* `email` is required and unique
* `age` must be at least 18

**Solution:**

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT CHECK (age >= 18)
);
```

### Exercise 3 — Hard

Create:

```text
customers
orders
```

Requirements:

* Every customer has a unique email.
* Customer name is required.
* Every order must belong to a customer.
* Order status defaults to `pending`.
* Status can only be `pending`, `paid`, or `cancelled`.
* Order total cannot be negative.

**Solution:**

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    total DECIMAL(10, 2) NOT NULL CHECK (total >= 0),
    status VARCHAR(20) DEFAULT 'pending'
        CHECK (status IN ('pending', 'paid', 'cancelled')),

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

---

# FAQ

**Q: What's the difference between `PRIMARY KEY` and `UNIQUE`?**

A: Both enforce uniqueness, but a primary key is the table's main row identifier and cannot be `NULL`. A table normally has one primary-key constraint but can have multiple `UNIQUE` constraints.

**Q: What's the difference between `NOT NULL` and `DEFAULT`?**

A: `NOT NULL` says a value is required. `DEFAULT` supplies a value when one isn't provided.

```sql
name VARCHAR(100) NOT NULL
```

versus:

```sql
country VARCHAR(100) DEFAULT 'Pakistan'
```

They solve different problems.

**Q: Can I have multiple `UNIQUE` constraints?**

A: Yes.

```sql
username VARCHAR(50) UNIQUE,
email VARCHAR(255) UNIQUE
```

Both can independently enforce uniqueness.

**Q: Can a table have multiple primary keys?**

A: It can have only one primary-key constraint, but that primary key can contain multiple columns.

```sql
PRIMARY KEY (order_id, product_id)
```

This is called a **composite primary key**.

**Q: What happens if I insert invalid data?**

A: The database rejects the operation and returns a constraint-violation error. The exact error message depends on the database system.

**Q: Can I add constraints after creating a table?**

A: Yes. For example:

```sql
ALTER TABLE users
ADD CONSTRAINT uq_users_email
UNIQUE (email);
```

You can also add foreign keys, checks, and other constraints using `ALTER TABLE`, subject to your database system's syntax.

**Q: Should validation happen in SQL or application code?**

A: Usually both. Application validation gives users useful feedback, while database constraints provide a final integrity boundary so invalid data cannot enter through another application, script, import, or service.

---

# Key Takeaways

* **Constraints are database-enforced rules for data integrity.**
* `PRIMARY KEY` → uniquely identifies rows.
* `FOREIGN KEY` → maintains relationships between tables.
* `NOT NULL` → requires a value.
* `UNIQUE` → prevents duplicate values.
* `CHECK` → validates conditions.
* `DEFAULT` → provides a value automatically.

# Next Steps

**Prerequisites:** `CREATE TABLE` → `PRIMARY KEY` → `FOREIGN KEY` → `Relationships`

**Next concepts:** `ALTER TABLE` → `Composite Keys` → `Referential Integrity` → `ON DELETE / ON UPDATE` → `Indexes` → `Transactions`
