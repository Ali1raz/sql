# SQL TRIGGERS — PostgreSQL
## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Beginner Example — Automatically Set `created_at`](#beginner-example-automatically-set-created_at)


## Quick Definition

A **trigger** is an automatic action that PostgreSQL runs when a specific event happens on a table.

Unlike a stored procedure, you don't manually call a trigger. PostgreSQL fires it automatically when something like `INSERT`, `UPDATE`, or `DELETE` happens.

Mental model:

```text
INSERT / UPDATE / DELETE
          ↓
       TRIGGER
          ↓
  Automatically run logic
```

A common use case is keeping an audit history:

```text
User changes email
       ↓
users UPDATE
       ↓
trigger fires
       ↓
user_changes gets a record
```

---

## The Basic Idea

PostgreSQL triggers normally work with two parts:

1. A **trigger function** — contains the logic.
2. A **trigger** — tells PostgreSQL when to execute that function.

So the pattern is:

```text
Trigger Function
      ↑
      │ executes
      │
   Trigger
      ↑
      │ fires on
      │
INSERT / UPDATE / DELETE
```

---

# Syntax

First create the trigger function:

```sql
CREATE OR REPLACE FUNCTION function_name()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- logic here

    RETURN NEW;
END;
$$;
```

Then create the trigger:

```sql
CREATE TRIGGER trigger_name
BEFORE INSERT ON table_name
FOR EACH ROW
EXECUTE FUNCTION function_name();
```

The important pieces:

`BEFORE` → run before the operation.

`AFTER` → run after the operation.

`INSERT` / `UPDATE` / `DELETE` → event that activates the trigger.

`FOR EACH ROW` → run once for every affected row.

`EXECUTE FUNCTION` → specifies the trigger function to run.

---

# Examples

## Beginner Example — Automatically Set `created_at`

Suppose you have:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    created_at TIMESTAMP
);
```

Create the trigger function:

```sql
CREATE OR REPLACE FUNCTION set_created_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.created_at = CURRENT_TIMESTAMP;

    RETURN NEW;
END;
$$;
```

Create the trigger:

```sql
CREATE TRIGGER users_created_at
BEFORE INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION set_created_at();
```

Now insert:

```sql
INSERT INTO users (name)
VALUES ('Ali');
```

You don't need to provide `created_at`.

The trigger automatically sets it.

```sql
SELECT *
FROM users;
```

Possible result:

```text
id | name | created_at
---+------+---------------------
1  | Ali  | 2026-09-30 10:30:00
```

---

# `NEW` and `OLD`

These are important in PostgreSQL triggers.

For an `INSERT`:

```text
NEW → newly inserted row
```

For a `DELETE`:

```text
OLD → row being deleted
```

For an `UPDATE`:

```text
OLD → values before update
NEW → values after update
```

Example:

```sql
UPDATE users
SET name = 'Ahmed'
WHERE id = 1;
```

The trigger can access:

```text
OLD.name → 'Ali'
NEW.name → 'Ahmed'
```

---

# Intermediate Example — Audit Changes

Suppose you want to record whenever a user's name changes.

Create an audit table:

```sql
CREATE TABLE user_audit (
    id SERIAL PRIMARY KEY,
    user_id INT,
    old_name VARCHAR(100),
    new_name VARCHAR(100),
    changed_at TIMESTAMP
);
```

Create the trigger function:

```sql
CREATE OR REPLACE FUNCTION log_name_change()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO user_audit (
        user_id,
        old_name,
        new_name,
        changed_at
    )
    VALUES (
        OLD.id,
        OLD.name,
        NEW.name,
        CURRENT_TIMESTAMP
    );

    RETURN NEW;
END;
$$;
```

Create the trigger:

```sql
CREATE TRIGGER user_name_change
AFTER UPDATE OF name ON users
FOR EACH ROW
EXECUTE FUNCTION log_name_change();
```

Now:

```sql
UPDATE users
SET name = 'Ahmed'
WHERE id = 1;
```

The trigger automatically inserts an audit record.

Check it:

```sql
SELECT *
FROM user_audit;
```

Possible result:

```text
user_id | old_name | new_name | changed_at
--------+----------+----------+---------------------
1       | Ali      | Ahmed    | 2026-09-30 10:40:00
```

You didn't manually insert into `user_audit`. The trigger did it.

---

# Advanced Example — Prevent Invalid Data

Suppose product prices must never be negative.

Create a trigger function:

```sql
CREATE OR REPLACE FUNCTION validate_product_price()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.price < 0 THEN
        RAISE EXCEPTION 'Product price cannot be negative';
    END IF;

    RETURN NEW;
END;
$$;
```

Create the trigger:

```sql
CREATE TRIGGER check_product_price
BEFORE INSERT OR UPDATE OF price ON products
FOR EACH ROW
EXECUTE FUNCTION validate_product_price();
```

Now:

```sql
INSERT INTO products (name, price)
VALUES ('Laptop', -500);
```

PostgreSQL raises an error:

```text
Product price cannot be negative
```

The invalid row is rejected.

---

# `BEFORE` vs `AFTER`

This is one of the most important things to understand.

### `BEFORE`

Runs before the operation.

Useful for:

* Validating data
* Modifying `NEW`
* Setting default values
* Preventing an operation

Example:

```sql
BEFORE INSERT
```

```text
INSERT
 ↓
TRIGGER
 ↓
data inserted
```

### `AFTER`

Runs after the operation.

Useful for:

* Audit logging
* Recording changes
* Performing follow-up actions

Example:

```sql
AFTER UPDATE
```

```text
UPDATE
 ↓
data updated
 ↓
TRIGGER
```

---

# `FOR EACH ROW` vs `FOR EACH STATEMENT`

### `FOR EACH ROW`

Runs once for every affected row.

```sql
CREATE TRIGGER example_trigger
AFTER UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION example_function();
```

If you update 100 users:

```sql
UPDATE users
SET status = 'active';
```

The trigger can execute **100 times**.

### `FOR EACH STATEMENT`

Runs once for the entire SQL statement.

```sql
CREATE TRIGGER example_trigger
AFTER UPDATE ON users
FOR EACH STATEMENT
EXECUTE FUNCTION example_function();
```

The same update affects 100 users, but the trigger fires once.

For most beginner trigger examples, you'll encounter `FOR EACH ROW`.

---

# Common Mistakes

### Mistake #1: Thinking you call the trigger yourself

You don't do:

```sql
CALL user_name_change();
```

The trigger fires automatically when its event occurs.

---

### Mistake #2: Forgetting `RETURN NEW`

For a row-level trigger function, you generally need to return the appropriate row.

For example:

```sql
BEGIN
    NEW.name = UPPER(NEW.name);

    RETURN NEW;
END;
```

For a `DELETE` trigger, you'd normally return `OLD`.

---

### Mistake #3: Using a trigger when a constraint is enough

Suppose you want:

```text
price >= 0
```

A simple constraint is often clearer:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    price DECIMAL CHECK (price >= 0)
);
```

You don't need a trigger for every validation rule.

Use triggers when you actually need **automatic procedural behavior**.

---

### Mistake #4: Creating too many triggers

Triggers can make database behavior less obvious.

You might run:

```sql
UPDATE users
SET name = 'Ahmed'
WHERE id = 1;
```

But behind the scenes, several triggers could execute additional operations.

That's powerful, but it can make debugging difficult.

---

# Performance Notes

Triggers execute automatically, so their work becomes part of the original database operation.

For example:

```sql
UPDATE users
SET status = 'active';
```

If 50,000 rows are updated and you have a row-level trigger, that trigger may execute 50,000 times.

Therefore:

* Keep trigger logic small.
* Avoid expensive queries inside row-level triggers.
* Be careful with triggers that modify other tables.
* Watch for recursive trigger behavior.
* Test bulk `INSERT`/`UPDATE`/`DELETE` operations.

A trigger isn't automatically bad for performance. The problem is **unnecessary or expensive trigger logic**.

---

# Practice

### Exercise 1 — Easy

Create a trigger that automatically converts a user's name to uppercase before inserting.

Hint:

```sql
NEW.name = UPPER(NEW.name);
```

Solution:

```sql
CREATE OR REPLACE FUNCTION uppercase_name()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.name = UPPER(NEW.name);

    RETURN NEW;
END;
$$;

CREATE TRIGGER users_uppercase_name
BEFORE INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION uppercase_name();
```

---

### Exercise 2 — Medium

Create an audit table that records whenever an order's status changes.

You need:

```text
order_id
old_status
new_status
changed_at
```

Use:

```sql
OLD.status
NEW.status
```

---

### Exercise 3 — Hard

Create a trigger that prevents an account balance from becoming negative.

Hint:

```sql
IF NEW.balance < 0 THEN
    RAISE EXCEPTION 'Balance cannot be negative';
END IF;
```

---

# FAQ

**Q: What's the difference between a trigger and a stored procedure?**

A procedure is explicitly executed:

```sql
CALL my_procedure();
```

A trigger fires automatically when its configured database event occurs.

```text
Procedure → You call it
Trigger   → Database calls it
```

**Q: Can a trigger run on `INSERT`, `UPDATE`, and `DELETE`?**

Yes.

```sql
CREATE TRIGGER example
BEFORE INSERT OR UPDATE OR DELETE ON users
...
```

The trigger function needs to handle the relevant operation appropriately.

**Q: What is `NEW`?**

`NEW` represents the new row.

For example:

```sql
NEW.email
```

gets the email value of the new/updated row.

**Q: What is `OLD`?**

`OLD` represents the previous row.

It's particularly useful for `UPDATE` and `DELETE`.

```sql
OLD.email
```

gets the previous email value.

**Q: Can a trigger modify `NEW`?**

Yes, especially in a `BEFORE` trigger.

```sql
NEW.name = UPPER(NEW.name);
```

The modified `NEW` row is then used for the operation.

**Q: Can triggers be used for auditing?**

Yes. This is a common use case.

```text
UPDATE
  ↓
AFTER UPDATE trigger
  ↓
audit table
```

**Q: Are triggers always a good idea?**

No. They're useful when behavior truly belongs at the database level, but excessive trigger logic can make systems harder to understand and debug.

**Q: Can a trigger call a function?**

Yes. In PostgreSQL, triggers execute a **trigger function**:

```sql
CREATE TRIGGER my_trigger
AFTER INSERT ON users
FOR EACH ROW
EXECUTE FUNCTION my_trigger_function();
```

# Key Takeaways

* 🔹 A **trigger runs automatically** when a database event occurs.
* 🔹 PostgreSQL triggers use a **trigger function + trigger definition**.
* 🔹 `NEW` represents the new row; `OLD` represents the previous row.
* 🔹 `BEFORE` is useful for validation/modifying data; `AFTER` is useful for auditing and follow-up actions.
* 🔹 Don't use triggers for everything—simple `CHECK`, `UNIQUE`, or `NOT NULL` constraints are often better for simple rules.

# Next Steps

A useful PostgreSQL learning path from here is:

`Triggers` → **Transactions** → **ACID** → **Exceptions** → **Functions** → **Advanced PL/pgSQL**.
