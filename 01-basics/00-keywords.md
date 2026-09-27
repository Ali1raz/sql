# SQL Keywords

## Quick Definition

**SQL keywords** are reserved words that have a specific meaning in SQL. They tell the database what operation you want to perform, such as retrieving, filtering, sorting, inserting, updating, or deleting data.

Examples: `SELECT`, `FROM`, `WHERE`, `INSERT`, `UPDATE`, `DELETE`, `ORDER BY`, and `GROUP BY`.

### The Basic Idea

Think of SQL keywords as **commands or instructions** that form the structure of a SQL statement.

1. Basic querying**

```sql
SELECT
FROM
WHERE
```

Example:

```sql
SELECT name, email
FROM users
WHERE age > 18;
```

**2. Sorting & limiting**

```sql
ORDER BY
ASC
DESC
LIMIT
OFFSET
```

```sql
SELECT *
FROM users
ORDER BY age DESC
LIMIT 10;
```

**3. Filtering**

```sql
AND
OR
NOT
IN
BETWEEN
LIKE
IS NULL
IS NOT NULL
```

```sql
SELECT *
FROM users
WHERE city IN ('Lahore', 'Karachi')
  AND age BETWEEN 18 AND 30;
```

**4. Creating/modifying data**

```sql
INSERT INTO
VALUES
UPDATE
SET
DELETE
```

```sql
INSERT INTO users (name, email)
VALUES ('Ali', 'ali@example.com');

UPDATE users
SET name = 'Raza'
WHERE id = 1;

DELETE FROM users
WHERE id = 1;
```

**5. Aggregation**

```sql
COUNT
SUM
AVG
MIN
MAX
GROUP BY
HAVING
```

```sql
SELECT city, COUNT(*)
FROM users
GROUP BY city
HAVING COUNT(*) > 5;
```

**6. Combining tables**

```sql
JOIN
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL JOIN
ON
```

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders
ON users.id = orders.user_id;
```

**7. Database/table structure**

```sql
CREATE DATABASE
CREATE TABLE
ALTER TABLE
DROP TABLE
TRUNCATE
```

**8. Constraints**

```sql
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
DEFAULT
CHECK
REFERENCES
```

Example:

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INT CHECK (age >= 18),
    country VARCHAR(50) DEFAULT 'Pakistan'
);
```

**9. More advanced — learn later**

```sql
CASE
AS
DISTINCT
UNION
UNION ALL
EXISTS
WITH
```

A good learning progression is:

`SELECT → FROM → WHERE → ORDER BY → LIMIT → INSERT → UPDATE → DELETE → GROUP BY → HAVING → JOIN → subqueries → CTEs → indexes → transactions`

One blind spot: **SQL keywords aren't the main thing to memorize.** Learn what each clause does and practice combining them. For backend development, being able to write a correct `JOIN + WHERE + GROUP BY` query matters far more than knowing 100 keywords.



## Common Mistakes

### Mistake #1: Using `WHERE` after `ORDER BY`

Incorrect:

```sql
SELECT name
FROM users
ORDER BY name
WHERE country = 'Pakistan';
```

Correct:

```sql
SELECT name
FROM users
WHERE country = 'Pakistan'
ORDER BY name;
```

SQL keywords generally follow a particular statement structure.

### Mistake #2: Confusing `WHERE` and `HAVING`

`WHERE` filters individual rows:

```sql
SELECT *
FROM users
WHERE country = 'Pakistan';
```

`HAVING` filters groups:

```sql
SELECT country, COUNT(*)
FROM users
GROUP BY country
HAVING COUNT(*) > 5;
```

### Mistake #3: Forgetting that `NULL` is special

Don't use:

```sql
WHERE email = NULL;
```

Use:

```sql
WHERE email IS NULL;
```
