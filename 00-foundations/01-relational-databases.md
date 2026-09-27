# Relational Databases

## Quick Definition

A **relational database** stores data in **tables** made up of rows and columns. Tables can be connected to each other using relationships, which lets you organize and query related data efficiently.

Examples: **MySQL, PostgreSQL, SQL Server, Oracle Database**.

### The Basic Idea

Think of a relational database like a collection of organized Excel sheets, except built for applications, multiple users, large amounts of data, and reliable relationships between data.

For example, an online store might have:

**`users`**

| id | name | country  |
| -: | ---- | -------- |
|  1 | Ali  | Pakistan |
|  2 | Sara | UAE      |

**`orders`**

|  id | user_id | product | amount |
| --: | ------: | ------- | -----: |
| 101 |       1 | Laptop  |   1200 |
| 102 |       2 | Phone   |    800 |

The `user_id` in `orders` connects an order to a user.

That's the core idea:

**Tables → store data**
**Rows → individual records**
**Columns → properties of those records**
**Relationships → connect tables**

### Basic Terminology

**Database** — The overall container holding your data.

**Table** — A structured collection of related data.

**Row** — One individual record.

**Column** — One attribute/property of a record.

**Primary Key** — A column that uniquely identifies each row.

### Why "Relational"?

Because the data isn't isolated.

You might have:

```text
users
  |
  | user_id
  ↓
orders
  |
  | product_id
  ↓
products
```

One user can have many orders, and each order can contain products.

SQL lets you work with these relationships.

For example:

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

This combines information from the two related tables.

### The Main Relationship Types

**One-to-One**

One user → one profile.

```text
User 1 ─── Profile 1
```

**One-to-Many**

One user → many orders.

```text
User 1 ─── Order 1
        ├── Order 2
        └── Order 3
```

This is extremely common.

**Many-to-Many**

Many students → many courses.

```text
Students ←→ Courses
```

This usually requires a third table, such as `student_courses`.

### Why Relational Databases Matter

They help you avoid storing the same information repeatedly.

Instead of putting this into every order:

```text
101 | Ali | Pakistan | Laptop | 1200
102 | Ali | Pakistan | Mouse   | 50
103 | Ali | Pakistan | Keyboard| 80
```

you separate the information:

```text
users
1 | Ali | Pakistan

orders
101 | 1 | Laptop
102 | 1 | Mouse
103 | 1 | Keyboard
```

Now Ali's information is stored once, while his orders reference him.

This makes data **more organized, consistent, and easier to maintain**.

### Key Takeaways

* **Tables** store related data.
* **Rows** represent individual records.
* **Columns** describe those records.
* **Primary keys** uniquely identify rows.
* **Foreign keys** connect tables.
* **Relationships** allow related data to be stored separately but queried together.
* SQL is the language commonly used to work with relational databases.

### Next Steps

Before diving deep into SQL, get comfortable with these concepts in roughly this order:

**Database → Table → Row → Column → Primary Key → Foreign Key → Relationships → SQL queries**

Once those are solid, `SELECT`, `WHERE`, `JOIN`, `GROUP BY`, and the rest of SQL start making a lot more sense.
