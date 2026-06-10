# MySQL / MariaDB Cheat Sheet

Quick reference for connecting to and navigating MySQL/MariaDB databases.

---

## Connecting

```bash
# Basic connection
mysql -h TARGET -u USERNAME -p

# No password (just press Enter at prompt)
mysql -h TARGET -u root

# Skip TLS (common in lab environments)
mysql -h TARGET -u root --skip-ssl

# Specify password inline (not recommended but works)
mysql -h TARGET -u root -pPASSWORD

# Specify port (default is 3306)
mysql -h TARGET -u root -P 3306 --skip-ssl
```

**Common default credentials to try:**
- `root` / no password
- `root` / `root`
- `admin` / `admin`
- `admin` / no password

---

## Navigation

```sql
-- List all databases
SHOW DATABASES;

-- Select a database to use
USE database_name;

-- List tables in current database
SHOW TABLES;

-- Show current database
SELECT DATABASE();

-- Show current user
SELECT USER();

-- Show MySQL version
SELECT VERSION();
```

---

## Exploring Tables

```sql
-- Show table structure (columns, types)
DESCRIBE table_name;
SHOW COLUMNS FROM table_name;

-- Show all data in a table
SELECT * FROM table_name;

-- Show specific columns
SELECT username, password FROM users;

-- Limit results
SELECT * FROM table_name LIMIT 10;

-- Search for specific value
SELECT * FROM users WHERE username = 'admin';

-- Count rows
SELECT COUNT(*) FROM table_name;
```

---

## Finding Interesting Data

```sql
-- Find all tables across all databases
SELECT table_schema, table_name 
FROM information_schema.tables 
WHERE table_type = 'BASE TABLE';

-- Find tables with interesting names
SELECT table_schema, table_name 
FROM information_schema.tables 
WHERE table_name LIKE '%user%' 
   OR table_name LIKE '%pass%' 
   OR table_name LIKE '%admin%' 
   OR table_name LIKE '%cred%';

-- Find all columns containing password-related names
SELECT table_schema, table_name, column_name 
FROM information_schema.columns 
WHERE column_name LIKE '%password%' 
   OR column_name LIKE '%passwd%' 
   OR column_name LIKE '%hash%';
```

---

## User and Privilege Enumeration

```sql
-- List all users
SELECT user, host FROM mysql.user;

-- Show user privileges
SHOW GRANTS FOR 'root'@'localhost';

-- Current user privileges
SHOW GRANTS;

-- Check if file read/write is possible
SELECT @@global.secure_file_priv;
```

---

## File Operations (if privileges allow)

```sql
-- Read a file from the server
SELECT LOAD_FILE('/etc/passwd');

-- Write output to a file
SELECT 'data' INTO OUTFILE '/var/www/html/shell.php';
```

Only work if MySQL user has FILE privilege and `secure_file_priv` is not restricted.

---

## Exiting

```sql
exit
quit
\q
```

---

## From the Command Line

```bash
# Run a single query without interactive shell
mysql -h TARGET -u root --skip-ssl -e "SHOW DATABASES;"

# Run query against specific database
mysql -h TARGET -u root --skip-ssl -e "SELECT * FROM users;" database_name

# Dump entire database
mysqldump -h TARGET -u root --skip-ssl database_name > dump.sql
```

---

## Common Port

| Port | Service |
|---|---|
| 3306 | MySQL / MariaDB default |

---

## Pentest Notes

- `information_schema` contains metadata about every database, table, and column on the server — always explore it
- `mysql` database contains user credentials — check `SELECT user, password FROM mysql.user;`
- Password hashes can be cracked offline with hashcat or john
- FILE privilege = potential for reading sensitive server files or writing webshells
- No password on root = full access to everything on the server
