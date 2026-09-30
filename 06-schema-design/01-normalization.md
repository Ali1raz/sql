# NORMALIZATION

Normalization is the process of organizing data in a relational database to **reduce duplicate data, prevent inconsistencies, and make relationships between data clearer**.

The basic idea is simple: **store each piece of information in the right place, once, and connect related information using keys.**

### The Basic Idea

Imagine this table:

| order_id | customer_name | customer_email                          | product | product_price |
| -------- | ------------- | --------------------------------------- | ------- | ------------- |
| 101      | Ali           | [ali@email.com](mailto:ali@email.com)   | Laptop  | 1000          |
| 102      | Ali           | [ali@email.com](mailto:ali@email.com)   | Mouse   | 25            |
| 103      | Sara          | [sara@email.com](mailto:sara@email.com) | Laptop  | 1000          |

The problem is that Ali's information is repeated, and the Laptop price is repeated.

If Ali changes his email, you might have to update several rows. Miss one, and your database contains conflicting information.

Normalization separates these concerns:

```text
customers
---------
customer_id
name
email

products
--------
product_id
name
price

orders
------
order_id
customer_id

order_items
-----------
order_id
product_id
quantity
```

Now each fact has a proper home.

### Why Normalization Matters

Normalization is useful when designing relational databases because it helps:

* Reduce duplicate data
* Prevent update inconsistencies
* Prevent accidental data loss
* Keep tables focused on one type of information
* Make relationships easier to understand
* Maintain data integrity

Think of it like organizing a workshop. You don't keep ten copies of the same hammer in ten different drawers. You keep one hammer in the right place and record where it belongs.

---

### Normal Forms

Normalization is usually explained through **normal forms**. Each normal form introduces another rule for improving the structure of the database.

The commonly discussed forms are:

```text
1NF → 2NF → 3NF → BCNF → 4NF → 5NF
```

For most everyday application database design, **1NF, 2NF, and 3NF** are the most important to understand first.

---

### 1NF — First Normal Form

A table is in **1NF** when:

1. Each column contains a single value.
2. There are no repeating groups.
3. Each row can be uniquely identified.

Bad example:

```text
customers
------------------------------------------------
customer_id | name | phone_numbers
1           | Ali  | 03001234567, 03111234567
```

`phone_numbers` contains multiple values.

Better:

```text
customers
-------------------------
customer_id | name
1           | Ali

customer_phones
-------------------------
customer_id | phone
1           | 03001234567
1           | 03111234567
```

Each cell now contains **one value**.

---

### 2NF — Second Normal Form

A table is in **2NF** when:

* It is already in 1NF.
* Every non-key column depends on the **whole primary key**, not just part of it.

This matters mainly when a table has a **composite primary key**.

Consider:

```text
order_items
------------------------------------------------
order_id | product_id | product_name | quantity
101      | 1          | Laptop       | 2
101      | 2          | Mouse        | 1
102      | 1          | Laptop       | 1
```

Suppose the primary key is:

```text
(order_id, product_id)
```

`quantity` depends on both `order_id` and `product_id`.

But `product_name` depends only on `product_id`.

That's a **partial dependency**, so the table violates 2NF.

Separate the product information:

```text
products
-------------------------
product_id | product_name
1          | Laptop
2          | Mouse
```

And keep order-specific information here:

```text
order_items
--------------------------------
order_id | product_id | quantity
101      | 1          | 2
101      | 2          | 1
102      | 1          | 1
```

---

### 3NF — Third Normal Form

A table is in **3NF** when:

* It is already in 2NF.
* Non-key columns depend only on the primary key.
* A non-key column should not depend on another non-key column.

Consider:

```text
employees
------------------------------------------------
employee_id | employee_name | department_id | department_name
1           | Ali           | 10            | Engineering
2           | Sara          | 20            | Marketing
3           | John          | 10            | Engineering
```

The dependency is:

```text
employee_id
    ↓
department_id
    ↓
department_name
```

`department_name` depends on `department_id`, not directly on `employee_id`.

Separate it:

```text
employees
-------------------------------
employee_id | employee_name | department_id

departments
-----------------------------
department_id | department_name
```

Now the relationship is cleaner.

---

# Syntax

Normalization itself doesn't have a special SQL command.

You implement normalization by **designing separate tables and connecting them with primary and foreign keys**.

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Here:

* `customers` stores customer information.
* `orders` stores order information.
* `customer_id` connects them.
* The foreign key prevents an order from referencing a nonexistent customer.

---

# Examples

### Beginner Example — Removing Repeated Data

Bad design:

```text
orders
---------------------------------------------------------
order_id | customer_name | customer_email | product
101      | Ali           | ali@email.com   | Laptop
102      | Ali           | ali@email.com   | Mouse
103      | Sara          | sara@email.com  | Keyboard
```

Better design:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product VARCHAR(100),

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Now Ali's information exists once.

---

### Intermediate Example — Orders and Products

A realistic order can contain multiple products.

Instead of:

```text
orders
-------------------------------------------------------
order_id | customer | product1 | product2 | product3
101      | Ali      | Laptop   | Mouse    | Keyboard
```

Use separate tables:

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10, 2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,

    PRIMARY KEY (order_id, product_id),

    FOREIGN KEY (order_id)
        REFERENCES orders(order_id),

    FOREIGN KEY (product_id)
        REFERENCES products(product_id)
);
```

The relationships are now:

```text
customers
    ↓
  orders
    ↓
order_items
    ↓
products
```

This is a common relational database pattern.

---

### Advanced Example — 3NF Design

Suppose you start with:

```text
orders
----------------------------------------------------------------
order_id | customer_name | customer_email | city | country
```

You might notice that customer information belongs to the customer, not the order.

A normalized design could be:

```sql
CREATE TABLE countries (
    country_id INT PRIMARY KEY,
    country_name VARCHAR(100) NOT NULL
);

CREATE TABLE cities (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(100) NOT NULL,
    country_id INT NOT NULL,

    FOREIGN KEY (country_id)
        REFERENCES countries(country_id)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    customer_email VARCHAR(255) UNIQUE,
    city_id INT,

    FOREIGN KEY (city_id)
        REFERENCES cities(city_id)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,

    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Now:

```text
Country
   ↓
City
   ↓
Customer
   ↓
Order
```

Each table represents a specific type of thing.

---

# Common Mistakes

### Mistake #1 — Treating normalization as "make more tables"

Normalization does **not** mean splitting everything into separate tables.

Bad over-normalization can make a simple database unnecessarily complicated.

The goal is:

```text
Less duplication
+
Clear dependencies
+
Correct relationships
```

Not:

```text
Maximum number of tables
```

### Mistake #2 — Repeating the same data

Bad:

```text
orders
------------------------------------------------
order_id | customer_id | customer_name
101      | 1           | Ali
102      | 1           | Ali
103      | 1           | Ali
```

Better:

```text
customers
------------------------
customer_id | name
1           | Ali
```

```text
orders
------------------------
order_id | customer_id
101      | 1
102      | 1
103      | 1
```

### Mistake #3 — Forgetting partial dependencies

If you have a composite key:

```text
PRIMARY KEY (order_id, product_id)
```

ask:

> Does every non-key column depend on **both** columns?

If not, you may have a 2NF problem.

### Mistake #4 — Confusing normalization with performance optimization

Normalization primarily concerns **data structure and integrity**.

It does not automatically make every query faster.

Sometimes normalized databases require more joins. That's one reason real production systems sometimes deliberately use **denormalization**.

---

# Performance Notes

Normalization can improve performance indirectly because less duplicated data means:

* Smaller tables
* Less storage
* Fewer repeated updates
* Less risk of inconsistent data

But highly normalized designs can require more `JOIN`s.

For example:

```sql
SELECT
    o.order_id,
    c.name
FROM orders o
JOIN customers c
    ON c.customer_id = o.customer_id;
```

For large systems, indexes on foreign-key columns can be important.

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

The key trade-off is:

```text
Normalization
      ↓
Less duplication + stronger integrity
      ↓
Potentially more JOINs

Denormalization
      ↓
More duplicated data
      ↓
Potentially simpler/faster reads
```

Don't denormalize just because joins look annoying. Measure the actual workload first.

---

# Practice

### Exercise 1 — Easy

You have:

```text
students
---------------------------------------
student_id | name | courses
1          | Ali  | SQL, Python, Java
```

What normalization problem exists?

**Hint:** `courses` contains multiple values.

**Solution:** Move courses into a related table.

```text
students
----------------------
student_id | name

student_courses
----------------------
student_id | course
```

---

### Exercise 2 — Medium

You have:

```text
order_items
-------------------------------------------------
order_id | product_id | product_name | quantity
```

Primary key:

```text
(order_id, product_id)
```

Which column creates a potential 2NF problem?

**Answer:** `product_name`.

It depends on `product_id`, not the entire composite key.

Move it into:

```text
products
-------------------------
product_id | product_name
```

---

### Exercise 3 — Hard

Given:

```text
employees
-----------------------------------------------------
employee_id | employee_name | department_id | department_name
```

Normalize this into a 3NF design.

**Expected structure:**

```text
employees
-------------------------------
employee_id
employee_name
department_id

departments
-------------------------------
department_id
department_name
```

---

# FAQ

**Q: When should I use normalization?**

A: Usually when designing transactional relational databases where data integrity and avoiding duplication matter.

When to use this: customer, order, inventory, banking, and employee systems.

Quick example:

```text
customers → orders → order_items
```

**Q: What's the difference between 1NF, 2NF, and 3NF?**

A:

```text
1NF → One value per cell
2NF → No partial dependency
3NF → No transitive dependency
```

**Q: Does every database need to be fully normalized?**

A: No. Production systems may deliberately use denormalization for specific performance or reporting requirements.

**Q: Does normalization remove all duplicate data?**

A: No. Some repetition is necessary. For example, a foreign key such as `customer_id` naturally appears in many orders. The important point is that the customer's descriptive information isn't unnecessarily repeated.

**Q: Can normalization make queries slower?**

A: It can increase the number of joins required, which can affect read performance depending on the workload and database design.

**Q: Does normalization improve data integrity?**

A: Yes. Separating related entities and enforcing foreign-key relationships makes many types of inconsistent data harder to create.

**Q: Is normalization only about splitting tables?**

A: No. The important part is understanding **dependencies between attributes** and structuring tables around those dependencies.

**Q: What should I learn after normalization?**

A: Learn **primary keys, foreign keys, relationships, joins, indexes, and denormalization**. Those concepts make normalization much easier to apply in real database designs.

---

# Key Takeaways

* 🧠 **Normalization organizes data to reduce unnecessary duplication.**
* 🔑 **Primary and foreign keys connect normalized tables.**
* 📦 **1NF:** one value per cell.
* 🔗 **2NF:** non-key columns depend on the whole key.
* 🧩 **3NF:** non-key columns shouldn't depend on other non-key columns.
* ⚖️ Normalization improves integrity, but excessive normalization can create unnecessary joins.

# Next Steps

**Prerequisites:** Primary Keys → Foreign Keys → Relationships → ER Diagrams

**Next concepts:** `1NF → 2NF → 3NF` in depth → `Denormalization` → `Indexes` → `Joins` → Database Design
