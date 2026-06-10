# MERN Stack

**Category:** Technology Notes / Web Application Stacks  
**Relevance:** Extremely common in modern web apps, SaaS products, internal tools, APIs

---

## What Is the MERN Stack?

MERN is an acronym for four technologies that together form a complete web application stack:

| Letter | Technology | Role |
|---|---|---|
| **M** | MongoDB | Database |
| **E** | Express.js | Web server / API framework |
| **R** | React | Frontend UI |
| **N** | Node.js | JavaScript runtime (powers Express) |

The appeal is one language (JavaScript) across the entire stack — frontend, backend, and database queries all written in JS. This makes it the default choice for JavaScript-only development shops and is why it's everywhere.

---

## The Components in Detail

### Node.js
JavaScript runtime built on Chrome's V8 engine. Lets JavaScript run on a server instead of just in a browser. Everything else in the MERN stack runs on top of Node.js.

- Default install location: `/usr/bin/node` or via NodeSource PPA on Ubuntu
- Version check: `node --version`
- Package manager: npm (Node Package Manager)
- All dependencies stored in `node_modules/` folder
- Project config in `package.json`

**Security relevance:**
- Runs as a single process — prototype pollution affects the entire process
- npm packages are a major attack surface — supply chain attacks, malicious packages
- `npm audit` checks for known vulnerabilities in dependencies

### Express.js
Minimal web framework for Node.js. Handles HTTP requests, routing, and middleware. The most deployed Node.js web framework by a significant margin.

**Default behavior that matters for pentesting:**
- Sends `X-Powered-By: Express` header on every response unless disabled
- Default error format for unhandled routes: `Cannot GET /path` (plain text)
- Listens on port 3000 by default in development
- Port 5000 also common

**Middleware stack:**
Express processes every request through a chain of middleware functions. Authentication, session management, logging, and body parsing all happen in middleware. This is important because prototype pollution can bypass middleware-based auth checks.

**Common middleware:**
- `express-session` — session management, creates `connect.sid` cookie
- `body-parser` — parses JSON request bodies (built into Express 4.16+)
- `helmet` — sets security headers, disables X-Powered-By
- `cors` — handles cross-origin requests
- `passport` — authentication

**Fingerprinting Express:**
```bash
# Check headers
curl -I http://TARGET:3000/

# Check error format
curl http://TARGET:3000/nonexistent

# Look for session cookie
curl -c cookies.txt http://TARGET:3000/
cat cookies.txt
```

### MongoDB
NoSQL document database. Stores data as JSON-like documents (called BSON) instead of rows and columns. Collections instead of tables, documents instead of rows.

**Default port:** 27017

**Why it matters for pentesting:**
- NoSQL injection — MongoDB uses operators like `$where`, `$gt`, `$regex` instead of SQL. If user input reaches a query unsanitized, attackers can inject these operators
- Misconfigured MongoDB instances exposed to the internet with no auth — extremely common finding
- `mongoexport` can dump entire databases if you gain access

**NoSQL injection example:**
```json
// Normal login query
{"username": "admin", "password": "password123"}

// Injected query — $ne means "not equal", bypasses password check
{"username": "admin", "password": {"$ne": ""}}
```

**Fingerprinting MongoDB:**
```bash
nmap -p 27017 TARGET
# If open, try connecting
mongo TARGET:27017
```

### React
Frontend JavaScript library for building user interfaces. Runs entirely in the browser — not on the server. React itself isn't usually a direct attack vector but understanding it helps with recon.

**What React tells you:**
- Build files in `/_next/static/` → Next.js (React framework)
- Build files in `/static/js/` → Create React App
- `__REACT_DEVTOOLS_GLOBAL_HOOK__` in source → React confirmed
- Component names and API endpoints sometimes visible in minified JS

**Recon value:**
- React apps make API calls to backend endpoints — check the Network tab in browser dev tools to find all API endpoints
- Minified JS files sometimes contain hardcoded API keys, endpoints, and credentials
- Source maps (`.map` files) sometimes accidentally deployed to production — reveals original unminified source code

---

## Typical MERN Deployment Architecture

```
Internet
    ↓
Nginx (reverse proxy, port 80/443)
    ↓
Express.js (port 3000 or 5000)
    ↓
MongoDB (port 27017, localhost only in prod)
```

**In production:** Nginx sits in front, strips headers, handles TLS. Express not directly exposed.

**In misconfigured/dev environments:** Express directly exposed on port 3000. MongoDB sometimes accidentally exposed on 27017.

**Internal tools:** Often Express directly exposed with no reverse proxy, MongoDB accessible on the network.

---

## Pentesting MERN Applications

### Recon Checklist

```bash
# 1. Port scan
nmap -sV -p 3000,5000,8080,27017,443,80 TARGET

# 2. Fingerprint Express
curl -I http://TARGET:3000/
curl http://TARGET:3000/nonexistent

# 3. Get session cookie
curl -c cookies.txt http://TARGET:3000/

# 4. Check for MongoDB exposure
nmap -p 27017 TARGET
mongo TARGET:27017 --eval "db.adminCommand({listDatabases:1})"

# 5. Check headers
curl -v http://TARGET:3000/ 2>&1 | grep -i "x-powered\|server\|cookie\|set-cookie"

# 6. Find API endpoints via browser Network tab or JS analysis
```

### API Endpoint Discovery

MERN apps are API-driven. Finding the API surface is critical.

```bash
# Directory brute force
gobuster dir -u http://TARGET:3000 -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://TARGET:3000/api -w /usr/share/wordlists/dirb/common.txt

# Common MERN API patterns
/api/users
/api/user/profile
/api/user/update
/api/auth/login
/api/auth/register
/api/admin
/api/admin/users
/api/admin/flag
```

### Common Vulnerabilities in MERN Apps

| Vulnerability | Where It Lives | Impact |
|---|---|---|
| Prototype Pollution | Merge/update functions | Auth bypass, privilege escalation |
| NoSQL Injection | MongoDB queries | Auth bypass, data exfiltration |
| Mass Assignment | Object update endpoints | Privilege escalation |
| JWT Weaknesses | Authentication | Auth bypass, token forgery |
| IDOR | User data endpoints | Access other users' data |
| XSS | React rendering of user input | Session hijack, credential theft |
| CORS Misconfiguration | Express CORS middleware | Cross-origin data theft |
| Exposed .env files | Deployment mistakes | Credentials, API keys |
| Exposed source maps | Build configuration | Source code disclosure |

### JWT Analysis

MERN apps commonly use JWT for authentication instead of sessions. When you see a Bearer token in requests or a JWT in localStorage/cookies:

```bash
# Decode JWT (base64 - no verification)
echo "eyJ..." | cut -d'.' -f2 | base64 -d 2>/dev/null

# Or use jwt.io in browser
# Check: algorithm (none attack if alg:none accepted)
# Check: expiry
# Check: sensitive data in payload
# Check: weak secret (use hashcat to crack HS256)
```

**JWT attacks:**
- `alg:none` — change algorithm to none, remove signature
- Weak secret — brute force HS256 secret with hashcat/john
- RS256 to HS256 confusion — use public key as HMAC secret

---

## Security Headers to Check

Express apps without Helmet are missing these by default:

```bash
curl -I http://TARGET:3000/ | grep -i "helmet\|csp\|hsts\|x-frame\|x-content\|referrer"
```

Missing headers that matter:
- `Content-Security-Policy` — XSS mitigation
- `X-Frame-Options` — clickjacking protection
- `X-Content-Type-Options: nosniff` — MIME sniffing
- `Strict-Transport-Security` — HTTPS enforcement
- No `X-Powered-By` — framework disclosure (Helmet removes this)

If `X-Powered-By: Express` is present → Helmet is not installed → other security headers probably also missing.

---

## Environment Files

Node.js apps use `.env` files for configuration. These contain database credentials, API keys, JWT secrets, and other sensitive values. Accidentally exposed `.env` files are one of the most common and impactful findings.

```bash
# Always check these
curl http://TARGET/.env
curl http://TARGET:3000/.env
curl http://TARGET/.env.local
curl http://TARGET/.env.development
curl http://TARGET/.env.production

# Also check for exposed git
curl http://TARGET/.git/config
```

A `.env` file typically looks like:
```
MONGODB_URI=mongodb://user:password@localhost:27017/dbname
JWT_SECRET=supersecretkey123
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG
```

---

## Tools for MERN Pentesting

| Tool | Use |
|---|---|
| Burp Suite / Caido | Intercept and modify API requests |
| jwt.io | Decode and analyze JWTs |
| Postman / curl | Manually test API endpoints |
| mongosh | MongoDB shell client |
| gobuster / ffuf | API endpoint discovery |
| npm audit | Check for known vulnerable dependencies |
| retire.js | Scan JS files for vulnerable libraries |
| nodejsscan | Static analysis of Node.js source code |

---

## Quick Reference — Common Express Patterns

```javascript
// Route patterns to look for
app.get('/api/user/:id', ...)      // IDOR potential - try other IDs
app.post('/api/user/update', ...)   // Prototype pollution / mass assignment
app.post('/api/auth/login', ...)    // NoSQL injection, brute force
app.get('/api/admin/*', ...)        // Auth bypass target

// Dangerous functions to look for in source
JSON.parse(req.body)               // Input parsing
Object.assign(target, source)      // Mass assignment / prototype pollution
eval(userInput)                    // RCE
exec(userInput)                    // Command injection
db.collection.find(userInput)      // NoSQL injection
```
