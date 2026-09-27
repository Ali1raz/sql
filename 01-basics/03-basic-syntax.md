### SQL Basic Syntax

The basic structure of a SQL query is:

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

Example:

```sql
SELECT name, email
FROM users
WHERE age >= 18;
```

The main SQL syntax patterns you should learn are:

**1. SELECT — retrieve data**

```sql
SELECT * FROM users;

SELECT name, email
FROM users;
```

**2. WHERE — filter data**

```sql
SELECT *
FROM users
WHERE age > 18;
```

**3. ORDER BY — sort results**

```sql
SELECT *
FROM users
ORDER BY age DESC;
```

**4. INSERT — add data**

```sql
INSERT INTO users (name, email, age)
VALUES ('Ali', 'ali@example.com', 22);
```

**5. UPDATE — modify data**

```sql
UPDATE users
SET age = 23
WHERE id = 1;
```

**6. DELETE — remove data**

```sql
DELETE FROM users
WHERE id = 1;
```

**7. CREATE TABLE — create a table**

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    age INT
);
```

**8. ALTER TABLE — modify table structure**

```sql
ALTER TABLE users
ADD COLUMN phone VARCHAR(20);
```

**9. DROP TABLE — delete table**

```sql
DROP TABLE users;
```

**10. DISTINCT — remove duplicates**

```sql
SELECT DISTINCT city
FROM users;
```

**11. LIMIT — restrict results**

```sql
SELECT *
FROM users
LIMIT 10;
```

A simple way to remember the basic query flow:

```text
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

Not every query needs all of them. This is the core syntax you'll build almost everything else on.
