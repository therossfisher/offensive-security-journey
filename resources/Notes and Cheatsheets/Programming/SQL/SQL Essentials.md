# SQL Fundamentals

SQL (Structured Query Language) is used to interact with relational databases. As a security professional you'll use it constantly — reading logs in a SIEM, understanding SQL injection vulnerabilities, extracting data from compromised databases, and querying threat intelligence databases.

---

## Core Concepts

### Database Types

| Type | Structure | Best For | Examples |
|---|---|---|---|
| Relational (SQL) | Tables with rows/columns, strict schema | Consistent structured data, accuracy critical | MySQL, PostgreSQL, MariaDB, SQLite |
| Non-relational (NoSQL) | Flexible, document/key-value based | Variable format data, scale | MongoDB, Redis, Elasticsearch |

### Key Terms

- **Table** — stores data in rows and columns, like a spreadsheet
- **Row** — a single record in a table
- **Column** — a field/attribute, has a defined data type
- **Primary Key** — unique identifier for each row in a table, no duplicates allowed, one per table
- **Foreign Key** — a column that references the primary key of another table, creates relationships between tables
- **DBMS** — Database Management System, the software interface between user and database (MySQL, MongoDB etc.)

### Data Types

```sql
INT           -- whole numbers
VARCHAR(255)  -- variable length text, max chars in parentheses
TEXT          -- long text
DATE          -- date (YYYY-MM-DD)
FLOAT         -- decimal numbers
BOOLEAN       -- true/false
```

---

## Getting Into MySQL

```bash
# Connect to MySQL
mysql -u root -p
mysql -u username -p -h HOST_IP    # remote connection

# Common default creds to try on CTF boxes
mysql -u root -p                    # blank password or 'root'
mysql -u admin -p
```

Once inside:

```sql
-- List all databases
SHOW DATABASES;

-- Select a database to work with
USE database_name;

-- List tables in current database
SHOW TABLES;

-- Describe table structure (columns, types, keys)
DESCRIBE table_name;
DESC table_name;        -- shorthand
```

---

## Database & Table Management

```sql
-- Create database
CREATE DATABASE database_name;

-- Delete database
DROP DATABASE database_name;

-- Create table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_date DATE
);

-- Add a column to existing table
ALTER TABLE users ADD last_login DATE;

-- Delete table
DROP TABLE table_name;
```

---

## CRUD Operations

The four fundamental database operations.

### CREATE — INSERT

```sql
-- Insert a single record
INSERT INTO users (username, password, email)
VALUES ("admin", "hashedpassword", "admin@example.com");

-- Insert multiple records
INSERT INTO users (username, password)
VALUES ("user1", "hash1"), ("user2", "hash2");
```

### READ — SELECT

```sql
-- Select all columns, all rows
SELECT * FROM users;

-- Select specific columns
SELECT username, email FROM users;

-- Filter with WHERE
SELECT * FROM users WHERE username = "admin";

-- Limit results
SELECT * FROM users LIMIT 10;

-- Select with multiple conditions
SELECT * FROM users WHERE username = "admin" AND email LIKE "%@example.com";
```

### UPDATE

```sql
-- Always use WHERE or you update every row
UPDATE users
SET password = "newhashedpassword"
WHERE username = "admin";

-- Update multiple columns
UPDATE users
SET password = "newhash", email = "new@email.com"
WHERE id = 1;
```

### DELETE

```sql
-- Always use WHERE or you delete everything
DELETE FROM users WHERE id = 1;

-- Delete all rows (keeps table structure)
DELETE FROM users;
```

---

## Clauses

### WHERE
Filter rows based on a condition.

```sql
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE username != "admin";
SELECT * FROM users WHERE created_date > "2024-01-01";
```

### DISTINCT
Return only unique values, remove duplicates.

```sql
SELECT DISTINCT category FROM tools;
```

### ORDER BY
Sort results ascending or descending.

```sql
SELECT * FROM tools ORDER BY name ASC;     -- A to Z
SELECT * FROM tools ORDER BY amount DESC;  -- highest first
SELECT * FROM tools ORDER BY name ASC LIMIT 1;  -- first alphabetically
```

### GROUP BY
Group rows with the same value, used with aggregate functions.

```sql
SELECT category, COUNT(*) FROM tools GROUP BY category;
```

### HAVING
Filter after grouping (WHERE filters before grouping, HAVING filters after).

```sql
SELECT category, COUNT(*)
FROM tools
GROUP BY category
HAVING COUNT(*) > 2;

-- With LIKE
SELECT name, COUNT(*)
FROM books
GROUP BY name
HAVING name LIKE '%Hack%';
```

---

## Operators

### Logical Operators

```sql
-- LIKE — pattern matching, % is wildcard
WHERE description LIKE "%exploit%"          -- contains "exploit"
WHERE name LIKE "Metasploit%"               -- starts with "Metasploit"
WHERE name LIKE "%framework"                -- ends with "framework"
WHERE name NOT LIKE "%0"                    -- doesn't end with 0

-- AND — both conditions must be true
WHERE category = "Multi-tool" AND description LIKE "%pentesters%"

-- OR — at least one condition must be true
WHERE name LIKE "%Android%" OR name LIKE "%iOS%"

-- NOT — reverse a condition
WHERE NOT category = "Offensive Security"

-- BETWEEN — value within a range (inclusive)
WHERE id BETWEEN 2 AND 4
WHERE published_date BETWEEN "2020-01-01" AND "2022-01-01"
```

### Comparison Operators

```sql
=       -- equal to
!=      -- not equal to
<       -- less than
>       -- greater than
<=      -- less than or equal to
>=      -- greater than or equal to
```

---

## Functions

### String Functions

```sql
-- LENGTH — character count of a string
SELECT name, LENGTH(name) AS name_length FROM tools;
SELECT name, LENGTH(name) AS len FROM tools ORDER BY len DESC LIMIT 1;  -- longest name

-- CONCAT — combine strings
SELECT CONCAT(name, " costs $", amount) AS info FROM tools;

-- GROUP_CONCAT — combine values from multiple rows into one string
SELECT GROUP_CONCAT(name SEPARATOR ' & ') FROM tools WHERE amount NOT LIKE '%0';
SELECT category, GROUP_CONCAT(name SEPARATOR ", ") FROM tools GROUP BY category;

-- SUBSTRING — extract part of a string
SELECT SUBSTRING(published_date, 1, 4) AS year FROM books;  -- extract year
```

### Aggregate Functions

```sql
-- COUNT — number of rows
SELECT COUNT(*) FROM tools;
SELECT COUNT(*) AS total FROM tools WHERE category = "Network";

-- SUM — total of a numeric column
SELECT SUM(amount) FROM tools;

-- MAX / MIN — highest or lowest value
SELECT MAX(amount) FROM tools;
SELECT MIN(published_date) AS earliest FROM books;

-- AVG — average value
SELECT AVG(amount) FROM tools;
```

---

## Security Relevance

### SQL Injection Basics
SQL injection happens when user input is passed directly into a query without sanitization. Understanding SQL syntax is prerequisite knowledge for understanding SQLi.

```sql
-- Vulnerable query (conceptual — don't run)
SELECT * FROM users WHERE username = '$input' AND password = '$pass'

-- Classic SQLi payloads abuse the SQL syntax:
-- admin'--        comments out the password check
-- ' OR '1'='1    always true condition
-- ' OR 1=1--     bypass authentication
```

See: [[sql-injection-notes]]

### Things to Look For on Compromised Systems

```sql
-- Once you have DB access on a box, look for:
SHOW DATABASES;                              -- what databases exist
USE database_name;
SHOW TABLES;                                 -- what tables exist
SELECT * FROM users;                         -- user credentials
SELECT * FROM users LIMIT 5;                 -- preview data
SELECT username, password FROM users;        -- just creds
SELECT * FROM config;                        -- config/secrets tables
SELECT * FROM sessions;                      -- active sessions
```

### Finding Credentials in Databases

```sql
-- Common table names to check
SELECT * FROM users;
SELECT * FROM accounts;
SELECT * FROM admin;
SELECT * FROM members;
SELECT * FROM credentials;
SELECT * FROM config;
SELECT * FROM settings;
SELECT * FROM secrets;

-- Common column names for passwords
SELECT username, password FROM users;
SELECT username, passwd FROM users;
SELECT username, pass FROM users;
SELECT username, pwd FROM users;
SELECT username, hash FROM users;
```

---

## Quick Reference — Common Query Patterns

```sql
-- Find longest string in a column
SELECT name, LENGTH(name) AS len FROM table ORDER BY len DESC LIMIT 1;

-- Count distinct values
SELECT COUNT(DISTINCT category) FROM tools;

-- Sum a column
SELECT SUM(amount) FROM tools;

-- Group and concatenate
SELECT GROUP_CONCAT(name SEPARATOR ' & ') FROM tools WHERE condition;

-- Find records where field matches pattern
SELECT * FROM table WHERE column LIKE "%keyword%";

-- Sort and get first/last
SELECT * FROM table ORDER BY name ASC LIMIT 1;   -- first alphabetically
SELECT * FROM table ORDER BY name DESC LIMIT 1;  -- last alphabetically

-- Filter by range
SELECT * FROM table WHERE amount BETWEEN 100 AND 500;

-- Exclude a value
SELECT * FROM table WHERE category != "Excluded";
```

---

## MySQL Cheat Sheet — Pentesting Context

```bash
# Access MySQL on a target (post-exploitation)
mysql -u root -p
mysql -u root --password=""          # blank password
mysql -u root -p -h 127.0.0.1

# MySQL config file locations (look for creds)
/etc/mysql/my.cnf
/etc/my.cnf
~/.my.cnf
/var/www/html/config.php             # web app DB creds
/var/www/html/wp-config.php          # WordPress DB creds
```

```sql
-- Useful recon queries once inside
SELECT user();                        -- current DB user
SELECT @@version;                     -- MySQL version
SELECT @@datadir;                     -- where DB files are stored
SHOW GRANTS FOR 'root'@'localhost';  -- what privs current user has
SELECT * FROM mysql.user;            -- all DB users and password hashes
```

---

## Connection to Other Topics

- **SQL Injection** — exploiting unsanitized SQL queries in web apps → [[sql-injection-notes]]
- **Web App Pentesting** — databases back almost every web app you'll test → [[burpsuite-overview-notes]]
- **Post-exploitation** — after getting a shell, databases are high-value targets for credential harvesting
- **SIEM/Logging** — defensive SQL queries used to hunt through log databases
