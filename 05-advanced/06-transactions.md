# SQL TRANSACTIONS — PostgreSQL
## Table of Contents

- [Quick Definition](#quick-definition)
- [Beginner Example](#beginner-example)
- [Intermediate Example — Rollback](#intermediate-example-rollback)
- [Advanced Example — Savepoints](#advanced-example-savepoints)


## Quick Definition

A **transaction** is a group of SQL operations treated as **one unit of work**.

Either all required operations succeed, or you can roll them back so the database doesn't keep a partial result.

A classic example is transferring money:

```text
Account A: -$100
Account B: +$100
```

You don't want only the first operation to succeed. Both operations should succeed together.

Mental model:

```text
BEGIN
  ↓
Operation 1
  ↓
Operation 2
  ↓
Operation 3
  ↓
COMMIT
  ↓
Changes saved
```

If something goes wrong:

```text
BEGIN
  ↓
Operation 1
  ↓
Operation 2 ❌
  ↓
ROLLBACK
  ↓
Changes undone
```

---

# The Basic Idea

The main PostgreSQL transaction commands are:

```sql
BEGIN;
```

Start a transaction.

```sql
COMMIT;
```

Permanently save the changes.

```sql
ROLLBACK;
```

Undo changes made during the transaction.

Basic pattern:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If something goes wrong:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

-- Something went wrong

ROLLBACK;
```

The update is undone.

---

# Syntax

### Start

```sql
BEGIN;
```

You may also see:

```sql
START TRANSACTION;
```

### Save

```sql
COMMIT;
```

### Undo

```sql
ROLLBACK;
```

---

# Examples

## Beginner Example

Suppose:

```text
accounts

id | name | balance
---+------+--------
1  | Ali  | 5000
2  | Sara | 3000
```

Transfer 1000 from Ali to Sara:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

Final result:

```text
id | name | balance
---+------+--------
1  | Ali  | 4000
2  | Sara | 4000
```

Both updates are saved together.

---

## Intermediate Example — Rollback

Suppose you start a transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 999;

ROLLBACK;
```

If account `999` doesn't exist, you don't want to keep the first update.

`ROLLBACK` returns the transaction to its previous state.

You can verify:

```sql
SELECT *
FROM accounts;
```

Ali's balance will be back to the original value.

---

## Advanced Example — Savepoints

A **savepoint** lets you partially roll back a transaction.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

SAVEPOINT before_second_update;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

ROLLBACK TO SAVEPOINT before_second_update;

COMMIT;
```

The first update remains, while the second update is undone.

Mental model:

```text
BEGIN
  ↓
Update A
  ↓
SAVEPOINT
  ↓
Update B
  ↓
ROLLBACK TO SAVEPOINT
  ↓
Update A remains
Update B undone
  ↓
COMMIT
```

---

# ACID Properties

Transactions are commonly described using **ACID**.

### Atomicity

All operations are treated as one unit.

```text
Operation A ✓
Operation B ❌
      ↓
Rollback
```

You don't end up with half the transaction.

### Consistency

The transaction should leave the database following its rules and constraints.

For example:

```sql
CHECK (balance >= 0)
```

should still be respected.

### Isolation

Concurrent transactions should not improperly interfere with each other.

For example, two users updating the same account shouldn't accidentally overwrite each other's changes.

PostgreSQL provides transaction isolation levels for controlling this behavior.

### Durability

After `COMMIT`, the database is responsible for making the committed changes persistent even if the system later encounters a failure.

Quick memory trick:

```text
A → All or nothing
C → Valid database state
I → Transactions don't improperly interfere
D → Committed data persists
```

---

# Common Mistakes

### Mistake #1: Forgetting `COMMIT`

```sql
BEGIN;

UPDATE users
SET status = 'inactive'
WHERE id = 10;
```

Depending on your client/session, the transaction may remain open instead of being committed.

Finish it with:

```sql
COMMIT;
```

---

### Mistake #2: Using `ROLLBACK` after `COMMIT`

Once you've done:

```sql
COMMIT;
```

you cannot then do:

```sql
ROLLBACK;
```

to undo those already-committed changes.

`ROLLBACK` only affects the current uncommitted transaction.

---

### Mistake #3: Leaving transactions open

This is a real production problem.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

-- Connection sits here for a long time
```

An open transaction can hold locks and cause other queries to wait.

Keep transactions **short and focused**.

---

### Mistake #4: Assuming every SQL statement needs a manual transaction

PostgreSQL normally runs individual statements in their own transaction when you're not explicitly managing one.

So:

```sql
UPDATE users
SET status = 'active'
WHERE id = 1;
```

is effectively handled as a single transaction.

Explicit transactions become important when **multiple statements must succeed or fail together**.

---

# Performance Notes

Transactions aren't inherently slow, but long-running transactions can cause problems.

Avoid:

```text
BEGIN
  ↓
Database work
  ↓
Wait 30 seconds
  ↓
More work
  ↓
COMMIT
```

Prefer:

```text
BEGIN
  ↓
Required operations
  ↓
COMMIT
```

Long transactions can:

* Hold locks longer
* Increase contention
* Prevent cleanup of old row versions
* Increase resource usage

For PostgreSQL, keeping transactions short is especially important in busy production systems.

---

# Practice

### Exercise 1 — Easy

Transfer `500` from account `1` to account `2`.

Solution:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

COMMIT;
```

---

### Exercise 2 — Medium

Perform an update but decide not to keep it.

```sql
BEGIN;

UPDATE users
SET status = 'inactive'
WHERE id = 5;

ROLLBACK;
```

The user's status returns to its previous state.

---

### Exercise 3 — Hard

Use a savepoint:

1. Update account 1.
2. Create a savepoint.
3. Update account 2.
4. Roll back only the second update.
5. Commit the first update.

Solution:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

SAVEPOINT after_account_1;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

ROLLBACK TO SAVEPOINT after_account_1;

COMMIT;
```

---

# FAQ

**Q: What's the difference between `COMMIT` and `ROLLBACK`?**

`COMMIT` saves the transaction.

`ROLLBACK` undoes the uncommitted transaction.

```sql
COMMIT;    -- Save
ROLLBACK;  -- Undo
```

**Q: Why do I need transactions?**

When multiple operations must succeed together.

For example:

```text
Remove money from A
       +
Add money to B
```

You don't want only one operation to succeed.

**Q: Does `SELECT` need a transaction?**

A `SELECT` can run without you explicitly writing `BEGIN`, but PostgreSQL still executes statements within transaction semantics.

You usually don't need to manually create a transaction just for a simple `SELECT`.

**Q: What happens if a query fails inside a transaction?**

The transaction can enter an aborted state. You generally need to roll it back before continuing with normal commands.

```sql
BEGIN;

UPDATE users
SET status = 'active'
WHERE id = 1;

-- Some statement fails

ROLLBACK;
```

**Q: What is a savepoint?**

A savepoint creates a point you can roll back to without cancelling the entire transaction.

```sql
SAVEPOINT my_point;

ROLLBACK TO SAVEPOINT my_point;
```

**Q: Are transactions only for banking systems?**

No. They're useful anywhere multiple database operations must be treated as one unit.

Examples:

* Creating an order
* Updating inventory
* Processing payments
* Registering users
* Moving data between tables

**Q: What's a long-running transaction?**

A transaction that stays open for a significant amount of time.

These can cause locking and database-maintenance problems, so production transactions should generally be kept short.

---

# Key Takeaways

* 🔹 A **transaction groups SQL operations into one unit of work**.
* 🔹 `BEGIN` starts it, `COMMIT` saves it, and `ROLLBACK` undoes it.
* 🔹 Transactions are essential when multiple operations must succeed or fail together.
* 🔹 **ACID** describes important transaction properties: Atomicity, Consistency, Isolation, Durability.
* 🔹 Keep production transactions **short** to reduce locking and contention.

# Next Steps

A good PostgreSQL sequence from here is:

`Transactions` → **Isolation Levels** → **Locks & Concurrency** → **ACID in depth** → **Deadlocks** → **Error Handling in PL/pgSQL**.
