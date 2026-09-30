# STORED PROCEDURES IN PostgreSQL
## Table of Contents

- [Quick Definition](#quick-definition)
- [Beginner Example](#beginner-example)
- [Intermediate Example](#intermediate-example)
- [Advanced Example — Validation](#advanced-example-validation)


## Quick Definition

A **stored procedure** in PostgreSQL is a named set of SQL/PL/pgSQL statements stored inside the database. You can call the procedure whenever you need that logic.

Think of it as: **put repeated database operations into a reusable function-like command.**

PostgreSQL uses `CREATE PROCEDURE` for procedures and `CALL` to execute them.

### The Basic Idea

Instead of repeatedly writing:

```sql
UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;
```

you can create a procedure:

```sql
CREATE PROCEDURE transfer_money(
    sender_id INT,
    receiver_id INT,
    amount DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE accounts
    SET balance = balance - amount
    WHERE id = sender_id;

    UPDATE accounts
    SET balance = balance + amount
    WHERE id = receiver_id;
END;
$$;
```

Then call it:

```sql
CALL transfer_money(1, 2, 1000);
```

---

# Syntax

```sql
CREATE PROCEDURE procedure_name(parameter_name data_type)
LANGUAGE plpgsql
AS $$
BEGIN
    -- SQL statements
END;
$$;
```

Calling it:

```sql
CALL procedure_name(value);
```

For example:

```sql
CREATE PROCEDURE deactivate_user(user_id INT)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE users
    SET status = 'inactive'
    WHERE id = user_id;
END;
$$;
```

Execute:

```sql
CALL deactivate_user(5);
```

### Breaking it down

`CREATE PROCEDURE` → creates the procedure.

`deactivate_user` → procedure name.

`user_id INT` → input parameter.

`LANGUAGE plpgsql` → tells PostgreSQL we're using PL/pgSQL.

`$$ ... $$` → contains the procedure body.

`BEGIN ... END` → block containing the statements.

`CALL` → executes the procedure.

---

# Examples

## Beginner Example

Create a procedure to deactivate a user:

```sql
CREATE PROCEDURE deactivate_user(user_id INT)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE users
    SET status = 'inactive'
    WHERE id = user_id;
END;
$$;
```

Run it:

```sql
CALL deactivate_user(10);
```

Check the result:

```sql
SELECT *
FROM users
WHERE id = 10;
```

---

## Intermediate Example

A procedure can accept multiple parameters.

```sql
CREATE PROCEDURE update_user_country(
    user_id INT,
    new_country VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE users
    SET country = new_country
    WHERE id = user_id;
END;
$$;
```

Call:

```sql
CALL update_user_country(5, 'Pakistan');
```

The procedure performs the update using the supplied values.

---

## Advanced Example — Validation

PL/pgSQL allows procedural logic such as `IF`.

```sql
CREATE PROCEDURE withdraw_money(
    account_id INT,
    amount DECIMAL
)
LANGUAGE plpgsql
AS $$
DECLARE
    current_balance DECIMAL;
BEGIN
    SELECT balance
    INTO current_balance
    FROM accounts
    WHERE id = account_id;

    IF current_balance < amount THEN
        RAISE EXCEPTION 'Insufficient balance';
    END IF;

    UPDATE accounts
    SET balance = balance - amount
    WHERE id = account_id;
END;
$$;
```

Call:

```sql
CALL withdraw_money(1, 500);
```

If the account doesn't have enough money, PostgreSQL raises an exception instead of performing the withdrawal.

---

# Common Mistakes

### Mistake #1: Forgetting `CALL`

Creating the procedure doesn't execute it.

```sql
CREATE PROCEDURE deactivate_user(user_id INT)
...
```

You still need:

```sql
CALL deactivate_user(10);
```

---

### Mistake #2: Confusing procedure and function

PostgreSQL has both.

Procedure:

```sql
CREATE PROCEDURE ...
```

Called with:

```sql
CALL procedure_name(...);
```

Function:

```sql
CREATE FUNCTION ...
```

Usually called with:

```sql
SELECT function_name(...);
```

A simple mental model:

```text
Procedure → performs an operation
Function  → calculates/returns something
```

There is more overlap than that, but this distinction is useful when starting.

---

### Mistake #3: Forgetting `LANGUAGE plpgsql`

For a PL/pgSQL procedure:

```sql
CREATE PROCEDURE test()
AS $$
BEGIN
    ...
END;
$$;
```

Better:

```sql
CREATE PROCEDURE test()
LANGUAGE plpgsql
AS $$
BEGIN
    ...
END;
$$;
```

---

### Mistake #4: No validation

This is dangerous:

```sql
UPDATE accounts
SET balance = balance - amount
WHERE id = account_id;
```

If `amount` is negative, the operation could behave unexpectedly.

Validate inputs when the business rule requires it:

```sql
IF amount <= 0 THEN
    RAISE EXCEPTION 'Amount must be greater than zero';
END IF;
```

---

# Performance Notes

Stored procedures can reduce repeated application-side database logic and can keep related operations close to the data.

However, **putting SQL inside a procedure does not automatically make it faster**.

Performance still depends on:

* indexes
* query design
* joins
* amount of data processed
* locks
* transaction design
* execution plans

For example, if your procedure repeatedly searches:

```sql
SELECT *
FROM users
WHERE email = user_email;
```

an appropriate index can still matter:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

---

# Practice

### Exercise 1 — Easy

Create a procedure that changes a user's status.

Expected call:

```sql
CALL change_status(5, 'inactive');
```

Solution:

```sql
CREATE PROCEDURE change_status(
    user_id INT,
    new_status VARCHAR
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE users
    SET status = new_status
    WHERE id = user_id;
END;
$$;
```

### Exercise 2 — Medium

Create a procedure that increases a product's price by a given amount.

Hint:

```sql
UPDATE products
SET price = price + amount
...
```

Solution:

```sql
CREATE PROCEDURE increase_price(
    product_id INT,
    amount DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE products
    SET price = price + amount
    WHERE id = product_id;
END;
$$;
```

### Exercise 3 — Hard

Create a procedure that transfers money between two accounts and rejects negative/zero amounts.

Key ideas:

```sql
IF amount <= 0 THEN
    RAISE EXCEPTION 'Invalid amount';
END IF;
```

Then perform both `UPDATE` statements.

---

# FAQ

**Q: Why use a stored procedure?**

A: To keep reusable database operations inside PostgreSQL.

```sql
CALL deactivate_user(10);
```

Instead of sending the same SQL logic from your application repeatedly.

**Q: What's the difference between a procedure and a function in PostgreSQL?**

A: Procedures are executed with `CALL`; functions are generally used as expressions and can return values.

```sql
CALL my_procedure();
```

versus:

```sql
SELECT my_function();
```

**Q: Can a procedure have parameters?**

Yes.

```sql
CREATE PROCEDURE update_price(
    product_id INT,
    new_price DECIMAL
)
...
```

**Q: Can procedures contain `IF` statements?**

Yes. PL/pgSQL supports procedural logic:

```sql
IF amount > 1000 THEN
    RAISE NOTICE 'Large transaction';
END IF;
```

**Q: Can a procedure contain multiple SQL statements?**

Yes. That's one of their useful features.

```sql
BEGIN
    UPDATE ...;
    INSERT ...;
    DELETE ...;
END;
```

**Q: Can I return a value from a procedure?**

Procedures aren't designed like functions for returning a normal scalar result. If your main requirement is to calculate and return a value, a PostgreSQL **function** is usually the relevant feature.

**Q: Can I delete a procedure?**

Yes:

```sql
DROP PROCEDURE deactivate_user(INT);
```

PostgreSQL may need the parameter types to identify the exact procedure.

---

# Key Takeaways

* 🔹 PostgreSQL procedures are created with `CREATE PROCEDURE`.
* 🔹 Execute them with `CALL`.
* 🔹 `PL/pgSQL` lets you use SQL plus procedural logic such as `IF`, variables, and exceptions.
* 🔹 Procedures are useful for reusable database operations.
* 🔹 Don't confuse `PROCEDURE` with `FUNCTION`—they have different invocation and return semantics.

# Next Steps

A good PostgreSQL sequence from here is:

`Stored Procedures` → **Functions** → **PL/pgSQL variables & IF** → **Exceptions** → **Transactions** → **Triggers**.
