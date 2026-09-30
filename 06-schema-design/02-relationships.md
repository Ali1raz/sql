# DATABASE RELATIONSHIPS

## Quick Definition

A **database relationship** describes how rows in one table are connected to rows in another table.

In a relational database, relationships are usually created using **primary keys and foreign keys**.

The three relationships you need to know first are:

```text
One-to-One       1 : 1
One-to-Many      1 : N
Many-to-Many     M : N
```

---

### The Basic Idea

Think of tables as different groups of information.

For an e-commerce system:

```text
customers
    ↓
  orders
    ↓
order_items
    ↓
products
```

A customer can place many orders.

An order can contain many products.

A product can appear in many orders.

The relationships tell the database **how these tables are connected**.

---

# Primary Key and Foreign Key

Relationships are usually built using two important keys.

A **primary key** uniquely identifies a row.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);
```

A **foreign key** stores the primary key of a related row.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Here:

```text
customers.customer_id
        ↑
        |
orders.customer_id
```

`orders.customer_id` tells us which customer owns the order.

---

# 1. One-to-One Relationship

A **one-to-one relationship** means one row in table A is related to one row in table B.

Example:

```text
user → user_profile
```

One user has one profile.

```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_profiles (
    profile_id INT PRIMARY KEY,
    user_id INT UNIQUE,
    bio TEXT,

    FOREIGN KEY (user_id)
        REFERENCES users(user_id)
);
```

The important part is:

```sql
user_id INT UNIQUE
```

Without `UNIQUE`, multiple profiles could reference the same user.

Conceptually:

```text
users
1 ─────────── 1 user_profiles
2 ─────────── 1 user_profiles
3 ─────────── 1 user_profiles
```

### When is 1:1 useful?

Examples include:

```text
User → User Profile
Employee → Employee Details
User → Passport
Vehicle → Vehicle Registration
```

Sometimes you don't need a separate table for a 1:1 relationship. If the information is small and always accessed together, keeping it in one table may be simpler.

---

# 2. One-to-Many Relationship

This is the **most common relationship** you'll encounter.

One row in table A can have many related rows in table B.

Example:

```text
Customer → Orders
```

One customer can place many orders.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Example data:

```text
customers
---------------------
customer_id | name
1           | Ali
2           | Sara
```

```text
orders
---------------------
order_id | customer_id
101      | 1
102      | 1
103      | 1
104      | 2
```

The relationship is:

```text
Ali
 ├── Order 101
 ├── Order 102
 └── Order 103

Sara
 └── Order 104
```

Notice where the foreign key goes.

```text
customers                 orders
---------                 ------
customer_id  ←──────────  customer_id
                           FK
```

The foreign key goes on the **many side**.

That's a rule worth remembering:

> **In a one-to-many relationship, the foreign key normally belongs in the many-side table.**

---

# 3. Many-to-Many Relationship

A **many-to-many relationship** means many rows in table A can relate to many rows in table B.

Example:

```text
Students ↔ Courses
```

One student can take many courses.

One course can have many students.

You cannot properly represent this by putting one `course_id` directly inside `students`.

Instead, create a **junction table** (also called a bridge/intermediate table).

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE student_courses (
    student_id INT,
    course_id INT,

    PRIMARY KEY (student_id, course_id),

    FOREIGN KEY (student_id)
        REFERENCES students(student_id),

    FOREIGN KEY (course_id)
        REFERENCES courses(course_id)
);
```

The structure becomes:

```text
students
    ↓
student_courses
    ↑
courses
```

Example:

```text
students
----------------
1 | Ali
2 | Sara
```

```text
courses
----------------
10 | SQL
20 | Python
30 | Java
```

```text
student_courses
----------------
student_id | course_id
1          | 10
1          | 20
2          | 10
2          | 30
```

So:

```text
Ali  → SQL
Ali  → Python

Sara → SQL
Sara → Java
```

The junction table converts:

```text
Many-to-Many
     ↓
One-to-Many + One-to-Many
```

---

# Syntax

The basic relationship syntax is:

```sql
CREATE TABLE parent_table (
    id INT PRIMARY KEY
);

CREATE TABLE child_table (
    id INT PRIMARY KEY,
    parent_id INT,

    FOREIGN KEY (parent_id)
        REFERENCES parent_table(id)
);
```

Breaking it down:

```sql
FOREIGN KEY (parent_id)
```

Says:

> `parent_id` is a foreign key.

```sql
REFERENCES parent_table(id)
```

Says:

> It must reference an existing `id` in `parent_table`.

---

# Examples

## Beginner — Customer and Orders

Create the tables:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Insert data:

```sql
INSERT INTO customers (customer_id, name)
VALUES
    (1, 'Ali'),
    (2, 'Sara');

INSERT INTO orders (order_id, customer_id)
VALUES
    (101, 1),
    (102, 1),
    (103, 2);
```

Find each order's customer:

```sql
SELECT
    o.order_id,
    c.name
FROM orders o
JOIN customers c
    ON c.customer_id = o.customer_id;
```

Expected output:

```text
order_id | name
---------|------
101      | Ali
102      | Ali
103      | Sara
```

---

## Intermediate — Products and Order Items

An order can contain multiple products.

A product can appear in multiple orders.

Therefore:

```text
Orders ↔ Products
```

is a many-to-many relationship.

Create it like this:

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,

    PRIMARY KEY (order_id, product_id),

    FOREIGN KEY (order_id)
        REFERENCES orders(order_id),

    FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);
```

The important structure is:

```text
orders
   ↓
order_items
   ↑
products
```

`order_items` is the junction table.

---

## Advanced — Many-to-Many With Relationship Data

Here's where things get more interesting.

Suppose students enroll in courses.

You may also need to store:

* enrollment date
* grade
* status

Those attributes belong to the **relationship itself**, not directly to the student or course.

```sql
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    enrolled_at DATE NOT NULL,
    grade VARCHAR(2),
    status VARCHAR(20) DEFAULT 'active',

    PRIMARY KEY (student_id, course_id),

    FOREIGN KEY (student_id)
        REFERENCES students(student_id),

    FOREIGN KEY (course_id)
        REFERENCES courses(course_id)
);
```

Now the relationship isn't merely:

```text
Student → Course
```

It's:

```text
Student
   ↓
Enrollment
   ↓
Course
```

This pattern appears everywhere in real systems.

Examples:

```text
User → Role
Product → Category
Student → Course
Doctor → Patient
Order → Product
Employee → Project
```

---

# Common Mistakes

### Mistake #1 — Putting multiple IDs in one column

Bad:

```text
orders
-------------------------
order_id | product_ids
101      | 1,2,5
```

This creates multiple values inside one column and makes querying difficult.

Better:

```text
order_items
-------------------------
order_id | product_id
101      | 1
101      | 2
101      | 5
```

### Mistake #2 — Forgetting the foreign key

This:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT
);
```

stores the ID but doesn't enforce the relationship.

Better:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

### Mistake #3 — Putting the foreign key on the wrong side

For:

```text
Customer 1 ──── many Orders
```

the foreign key normally belongs in `orders`:

```text
orders.customer_id
```

not in `customers`.

### Mistake #4 — Forgetting `UNIQUE` in a one-to-one relationship

This:

```sql
user_id INT
```

allows multiple profiles for one user.

For a true 1:1 relationship:

```sql
user_id INT UNIQUE
```

---

# Performance Notes

Foreign keys establish relationships, but indexes help the database **find related rows efficiently**.

For example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

This can help queries such as:

```sql
SELECT *
FROM orders
WHERE customer_id = 1;
```

For junction tables, the primary key often provides a useful index:

```sql
PRIMARY KEY (student_id, course_id)
```

But remember that a composite index has an order. An index on:

```text
(student_id, course_id)
```

is not equivalent to having separate indexes for every possible lookup.

For large systems, index the columns that are frequently used for:

```text
JOIN
WHERE
ORDER BY
```

after checking the actual query workload.

---

# Practice

### Exercise 1 — Easy

Create a one-to-many relationship:

```text
Department → Employees
```

Requirements:

```text
departments
department_id
department_name

employees
employee_id
employee_name
department_id
```

**Solution:**

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);

CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT,

    FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);
```

---

### Exercise 2 — Medium

Create a many-to-many relationship:

```text
Authors ↔ Books
```

An author can write multiple books.

A book can have multiple authors.

**Hint:** You need three tables.

```text
authors
books
author_books
```

---

### Exercise 3 — Hard

Design an e-commerce relationship structure containing:

```text
Customers
Orders
Products
```

Requirements:

* One customer can have many orders.
* One order can contain many products.
* One product can appear in many orders.
* Each order item must store quantity.

**Expected relationship:**

```text
customers
    ↓ 1:N
orders
    ↓ 1:N
order_items
    N:1
products
```

---

# FAQ

**Q: What's the difference between a primary key and a foreign key?**

A: A primary key identifies a row in its own table. A foreign key points to a row in another table.

```sql
customer_id INT PRIMARY KEY
```

versus:

```sql
customer_id INT,
FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id)
```

**Q: Where does the foreign key go in a one-to-many relationship?**

A: Normally on the **many side**.

```text
Customer 1 ──── N Orders

orders.customer_id → customers.customer_id
```

**Q: How do I represent many-to-many relationships?**

A: Use a junction table.

```text
students
    ↓
student_courses
    ↑
courses
```

**Q: Can a foreign key reference a non-primary-key column?**

A: It can reference a column that has an appropriate uniqueness constraint, depending on the database system, but primary keys are the standard and safest choice for relationships.

**Q: What happens if I delete a parent row?**

A: The database may prevent the deletion if child rows reference it, unless you've defined an action such as `ON DELETE CASCADE`, `SET NULL`, or `RESTRICT`.

Example:

```sql
FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id)
    ON DELETE CASCADE
```

Use cascading deletes carefully. One delete can remove many related rows.

**Q: Why do many-to-many relationships need a junction table?**

A: Because a single foreign-key column can represent one related row per record. A junction table allows an unlimited number of connections between the two entities.

**Q: Is a junction table only for IDs?**

A: No. It can also contain information about the relationship itself.

```text
student_id
course_id
enrolled_at
grade
status
```

**Q: Can I have relationships without foreign-key constraints?**

A: Yes, technically. You can store matching IDs without declaring a foreign key. But then the database doesn't enforce referential integrity for you.

---

# Key Takeaways

* **1:1** → one row relates to one row.
* **1:N** → one row relates to many rows; the foreign key normally goes on the many side.
* **M:N** → use a **junction table**.
* **Primary keys identify; foreign keys connect.**
* Relationship tables can store additional information about the relationship itself.

# Next Steps

**Prerequisites:** Primary Keys → Foreign Keys → Normalization

**Next concepts:** `JOINs` → `INNER JOIN` → `LEFT JOIN` → `Many-to-Many Queries` → `Referential Integrity` → `ON DELETE / ON UPDATE`
