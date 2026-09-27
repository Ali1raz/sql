# SQL Operators

## Quick Definition

SQL operators are symbols or keywords used to perform operations on values and conditions. They are commonly used to compare values, perform calculations, combine conditions, and filter data.

## 1. Comparison operators

=       -- equal
<>      -- not equal
!=      -- not equal
>       -- greater than
<       -- less than
>=      -- greater than or equal
<=      -- less than or equal

**Example:**
```sql
SELECT *
FROM users
WHERE age >= 18;
```

## 2. Arithmetic operators

+       -- addition
-       -- subtraction
*       -- multiplication
/       -- division
%       -- remainder

**Example:**
```sql
SELECT price * quantity AS total
FROM orders;
```

## 3. Logical operators
```sql
AND
OR
NOT
```

**Example:**
```sql
SELECT *
FROM users
WHERE age >= 18
AND city = 'Lahore';
```

**Example:**
```sql
WHERE city IN ('Lahore', 'Karachi');

WHERE age BETWEEN 18 AND 30;

WHERE name LIKE 'A%';

WHERE email IS NULL;

LIKE patterns:

'A%'    → starts with A
'%a'    → ends with a
'%ali%' → contains ali
'_a%'   → second character is a
```

## 5. Concatenation

In PostgreSQL:

||

**Example:**

```sql
SELECT first_name || ' ' || last_name AS full_name
FROM users;
```

## 6. Common advanced operators

You'll encounter these later:

ANY
ALL
EXISTS

For SQL basics, your priority is:
```sql
= <> > < >= <= AND OR NOT IN BETWEEN LIKE IS NULL
```


Those are the ones you should be comfortable with before moving into JOIN, GROUP BY, and subqueries.

## Common Mistakes
**Mistake #1: Using = incorrectly**

For comparison, use:
```sql
WHERE age = 25;
```

Don't use:
```sql
WHERE age == 25;
```
== is supported by some database systems, but = is the standard SQL comparison operator.

**Mistake #2: Forgetting parentheses with AND and OR**

This:
```sql
WHERE country = 'Pakistan'
   OR country = 'India'
   AND age >= 18;
```
doesn't necessarily mean what you might expect because AND has higher precedence than OR.

Make the intention clear:
```sql
WHERE (country = 'Pakistan' OR country = 'India')
  AND age >= 18;
```

**Mistake #3: Comparing NULL with =**

Incorrect:
```sql
WHERE email = NULL;
```

Correct:
```sql
WHERE email IS NULL;
```

## FAQ

**Q: What's the difference between = and ==?**

A: = is the standard SQL equality operator. == is not standard SQL and should generally be avoided for portable SQL.

**Q: What's the difference between AND and OR?**
A: AND requires all conditions to be true. OR requires at least one condition to be true.

**Q: Is BETWEEN inclusive?**

A: Yes. Both boundary values are included.

```sql 
WHERE age BETWEEN 18 AND 30;
```

**Q: When should I use IN instead of multiple ORs?**

A: Use IN when checking the same column against multiple possible values.

**Q: How do I check for NULL?**

A: Use IS NULL or IS NOT NULL.

```sql
WHERE email IS NULL;
```

**Q: What does % mean with LIKE?**

A: It represents zero or more characters.

```sql
WHERE name LIKE 'A%';
```

finds names beginning with A.

**Q: Does operator order matter?**

A: Yes. SQL evaluates operators according to precedence rules. Use parentheses when you want to make the intended logic explicit.

```sql
WHERE (country = 'Pakistan' OR country = 'India')
  AND age >= 18;
```
