# SQL Data Types

## Table of Contents

- [Quick Definition](#quick-definition)
- [The Basic Idea](#the-basic-idea)
- [Common Mistakes](#common-mistakes)
- [Practice](#practice)
- [FAQ](#faq)
- [Key Takeaways](#key-takeaways)
- [Next Steps](#next-steps)

## Quick Definition

SQL data types define what kind of value a column can store. They tell the database whether a column should contain text, numbers, dates, true/false values, and so on.

For example, a user's name needs text, while their age needs a number.

## The Basic Idea

Think of a data type as a rule for what can go inside a column.

```
users
┌──────┬───────────┬─────┬─────────────┐
│ id   │ name      │ age │ created_at  │
├──────┼───────────┼─────┼─────────────┤
│ 1    │ Ali       │ 25  │ 2026-09-27  │
└──────┴───────────┴─────┴─────────────┘
 INT     VARCHAR     INT     DATE
```

Choosing the appropriate type helps keep your data valid and organized.

For SQL basics, learn data types in these groups:

**1. Numbers**

```sql
INT              -- whole numbers
BIGINT           -- very large whole numbers
DECIMAL(p,s)     -- exact decimal numbers
NUMERIC(p,s)     -- same idea as DECIMAL
FLOAT            -- approximate decimal numbers
```

Example:

```sql
age INT
salary DECIMAL(10,2)
```

`DECIMAL(10,2)` means **10 total digits, 2 after the decimal** → `12345678.90`.

**2. Text**

```sql
CHAR(n)          -- fixed-length text
VARCHAR(n)       -- variable-length text
TEXT             -- long text
```

Example:

```sql
name VARCHAR(100)
email VARCHAR(255)
description TEXT
```

Usually, `VARCHAR` is used for names, emails, titles, etc.

**3. Date & Time**

```sql
DATE             -- 2026-09-27
TIME             -- 14:30:00
TIMESTAMP        -- date + time
TIMESTAMPTZ      -- date + time + timezone
```

For PostgreSQL backend applications, `TIMESTAMPTZ` is often the safer choice for timestamps.

**4. Boolean**

```sql
BOOLEAN
```

Values:

```sql
TRUE
FALSE
```

Example:

```sql
is_active BOOLEAN DEFAULT TRUE
```

**5. JSON**
Especially useful in PostgreSQL:

```sql
JSON
JSONB
```
JSONB - stores JSON data in binary format (more efficient than TEXT).

Example:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  metadata JSONB
);

INSERT INTO users (metadata) VALUES (
  '{"role": "admin", "preferences": {"theme": "dark", "notifications": true}}'
);
```

```sql
-- Query JSON
SELECT metadata->'preferences'->>'theme' FROM users;
```

`JSONB` is generally preferred in PostgreSQL when you need to query/index JSON data.

**6. UUID**

```sql
UUID
```

Example:

```sql
id UUID PRIMARY KEY
```

Very common in modern backend applications.

**7. Binary**

```sql
BYTEA
```

## Common Mistakes
### Mistake #1: Using VARCHAR for everything

You could store an age as text:
```sql
age VARCHAR(10)
```
But this is a poor choice.

Better:

```sql
age INT
```

The database can then treat the value as a number.

### Mistake #2: Using FLOAT for money

Avoid:
```sql
price FLOAT
```

For monetary values, use:

```sql
price DECIMAL(10, 2)
```

### Mistake #3: Confusing NULL with 0

NULL means the value is missing or unknown.

0 is an actual numeric value.
```sql
age = 0       → the value is zero
age = NULL    → the age is unknown/missing
```

## Practice

### Exercise 1 — Easy

Choose the appropriate type for:
```sql 
name
age
price
birth_date
```

**Solution:**

```sql
name       → VARCHAR
age        → INT
price      → DECIMAL
birth_date → DATE
```

### Exercise 2 — Medium

Create a students table with:

```sql
ID
Name
Age
Email
Date of birth
```

**Solution:** 

```sql
CREATE TABLE students (
    id INT,
    name VARCHAR(100),
    age INT,
    email VARCHAR(255),
    birth_date DATE
);
```

### Exercise 3 — Hard

Create an orders table containing:

```sql
Order ID
Customer ID
Total price
Order date
Payment status
```

**Solution:**

```sql
CREATE TABLE orders (
    id INT,
    customer_id INT,
    total_price DECIMAL(10, 2),
    order_date DATE,
    is_paid BOOLEAN
);
```

## FAQ

Q: What's the difference between CHAR and VARCHAR?

A: CHAR is designed for fixed-length text, while VARCHAR is for variable-length text.

Q: What's the difference between INT and DECIMAL?

A: INT stores whole numbers, while DECIMAL stores exact decimal values.

Q: Should I use FLOAT or DECIMAL for money?

A: Usually DECIMAL, because it represents decimal values exactly.

Q: What data type should I use for names?

A: Usually VARCHAR.

name VARCHAR(100)

Q: What type should I use for a date?

A: Use DATE when you only need the date.

birth_date DATE

Q: What type should I use for true/false values?

A: Use BOOLEAN where supported.

is_active BOOLEAN

Q: Is NULL a data type?

A: No. NULL represents a missing or unknown value.

## Key Takeaways
Data types define what a column can store.

```sql
INT → whole numbers.
DECIMAL → exact decimal numbers.
VARCHAR → variable-length text.
DATE → dates.
BOOLEAN → true/false.
NULL means missing or unknown, not zero.
```

Choose a type based on the actual kind of data the column represents.

## Next Steps

Learn these first:
```
INT
DECIMAL
VARCHAR
CHAR
TEXT
DATE
TIME
DATETIME
BOOLEAN
NULL
```

Then move on to constraints, such as PRIMARY KEY, NOT NULL, UNIQUE, and FOREIGN KEY.
