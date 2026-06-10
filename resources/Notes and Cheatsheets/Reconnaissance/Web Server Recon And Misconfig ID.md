# Web Server Recon and Misconfiguration Identification

**THM Room:** Web Server Attacks I  
**Difficulty:** Medium  
**Path:** Jr Penetration Tester / Web Application Security  
**Completed:** June 2026

---

## Core Concept

Default configurations prioritize ease of deployment over security. Version disclosure, directory listings, and status pages are enabled by default for diagnostic convenience. Finding these doesn't indicate negligence — it indicates default settings were never reviewed. This is extremely common in real engagements.

---

## The Four Server Types

| Port | Server | Primary Fingerprint |
|---|---|---|
| 80 | Apache2 | `Server: Apache/X.X.XX (Ubuntu)` |
| 8000 | Python HTTP Server | `Server: SimpleHTTP/0.6 Python/3.X.X` |
| 3000 | Node.js Express | `X-Powered-By: Express` (no Server header) |
| 8080 | Nginx | `Server: nginx/X.XX.X (Ubuntu)` |

---

## Fingerprinting Commands

```bash
# Headers only (HEAD request)
curl -sI http://TARGET:PORT/

# Full response body (GET request)
curl -s http://TARGET:PORT/

# Check error format (fingerprints server type)
curl -s http://TARGET:PORT/nonexistent-page

# Quick fingerprint all common ports
for port in 80 8000 3000 8080; do
    echo "=== Port $port ==="
    curl -sI http://TARGET:$port/ | grep -iE "server|x-powered-by"
done
```

---

## Python HTTP Server (Port 8000)

**What it is:** Built-in Python server started with `python3 -m http.server 8000`

**Why it's dangerous:**
- No access control whatsoever
- No authentication
- Serves the entire working directory including dotfiles
- No configuration file or blocklist
- One mode: serve everything

**What to check:**

```bash
# View directory listing
curl -s http://TARGET:8000/

# Get .env file (credentials often here)
curl -s http://TARGET:8000/.env

# Download archives and inspect
curl -s http://TARGET:8000/backup.zip -o backup.zip
unzip backup.zip -d backup-contents/
cat backup-contents/db_dump.sql
```

**Key difference from Apache/Nginx:** Python serves dotfiles like `.env` as normal files. Apache and Nginx block them by default. If you see a Python HTTP server, always try `.env`, `.git/config`, `.htpasswd`.

**Finding significance:** No exploitation needed. The server is working as designed. The misconfiguration is that it's running somewhere it shouldn't be.

---

## Apache (Port 80)

**Version disclosure:**
```bash
curl -sI http://TARGET:80 | grep -i server
# Server: Apache/2.4.58 (Ubuntu)
# ServerTokens OS = default Ubuntu setting, includes OS label
```

**Directory listing (`Options +Indexes`):**
```bash
# Check known directories for listing
curl -s http://TARGET:80/files/
# Look for: <title>Index of /files</title>
```

**mod_status page:**
```bash
curl -s http://TARGET:80/server-status
```
Reveals: active connections, request paths, worker states, server version, start time. Misconfigured with `Require all granted` instead of `Require local`.

**Always check `/server-status` even on default-looking configs** — a `Require all granted` in any virtual host silently overrides the localhost restriction.

**Find unlinked files with Gobuster:**
```bash
gobuster dir -u http://TARGET:80 \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x bak,txt,html -t 20
```

Look for:
- `.bak` files — backup configs, often contain credentials
- `.htpasswd` — password hashes for Basic Auth
- Old source code copies

---

## Node.js Express (Port 3000)

**Fingerprinting:**
```bash
curl -sI http://TARGET:3000
# X-Powered-By: Express  ← primary fingerprint
# No Server header by default

curl -s http://TARGET:3000
# Often returns JSON: {"status":"ok","app":"name","version":"1.2.0"}
```

**Trigger verbose errors:**
```bash
curl -s http://TARGET:3000/api/users | python3 -m json.tool
```
Look for stack traces containing:
- Internal file paths (`/opt/nodeapp/app.js:16`)
- Database queries (`SELECT * FROM users`)
- Module paths revealing dependencies and versions

**Stack traces appear when:**
- `NODE_ENV` is set to `development`
- Developer wrote a custom error handler that wasn't hardened
- Custom handlers override `NODE_ENV=production` protection

**Debug route enumeration:**
```bash
curl -s http://TARGET:3000/api/routes
# Returns all registered routes if debug endpoint exists
# ["GET /", "GET /api/users", "GET /api/debug/env"]
```

**Environment variable exposure:**
```bash
curl -s http://TARGET:3000/api/debug/env
# May return: DB_PASSWORD, SECRET_KEY, NODE_ENV, API keys
```

`NODE_ENV=development` on a production server = deployed without hardening.

**Static files:**
```bash
curl -s http://TARGET:3000/static/config.js
# May contain: API endpoints, internal hostnames, debug flags
```

Note: `express.static()` blocks dotfiles by default. A 404 on `.env` doesn't mean it doesn't exist — middleware is blocking it.

**The Express investigation workflow:**
1. Headers → confirm framework
2. Trigger errors → see internals
3. Debug endpoint → enumerate all routes  
4. Environment endpoint → find credentials
5. Static files → find embedded config

---

## Nginx (Port 8080)

**Version disclosure:**
```bash
curl -sI http://TARGET:8080 | grep -i server
# Server: nginx/1.24.0 (Ubuntu)

# Version also appears in 404 error pages
curl -s http://TARGET:8080/nonexistent
# <center>nginx/1.24.0 (Ubuntu)</center>
```

`server_tokens off` suppresses version from both headers AND error pages simultaneously.

**Directory listing (autoindex):**
```bash
curl -s http://TARGET:8080/files/
# Nginx autoindex format: filename, date, size table
```

Nginx doesn't enable directory listing by default — requires `autoindex on` in config. When present, read everything.

**nginx_status endpoint:**
```bash
curl -s http://TARGET:8080/nginx_status
# Active connections: 1
# server accepts handled requests
#  15 15 15
# Reading: 0 Writing: 1 Waiting: 0
```

The three unlabeled numbers are: total accepted, total handled, total requests since start.

Misconfigured with `allow all` instead of `allow 127.0.0.1; deny all;`

Config location if you have shell: `/etc/nginx/sites-available/`

---

## Security Header Audit

Run against all ports at once:

```bash
for port in 80 8000 3000 8080; do
    echo "=== Port $port ==="
    curl -sI http://TARGET:$port/ | grep -iE "x-frame-options|x-content-type|content-security-policy|strict-transport|referrer-policy" || echo "(no security headers found)"
done
```

| Header | Protects Against | Example Value |
|---|---|---|
| `X-Frame-Options` | Clickjacking | `DENY` or `SAMEORIGIN` |
| `X-Content-Type-Options` | MIME sniffing | `nosniff` |
| `Content-Security-Policy` | XSS, resource injection | `default-src 'self'` |
| `Referrer-Policy` | Referrer data leakage | `no-referrer` |
| `Strict-Transport-Security` | Forces HTTPS | `max-age=31536000` |

**All four server types have NO security headers by default.** Security headers require active configuration.

Note: `X-Frame-Options` is superseded by `Content-Security-Policy: frame-ancestors`. Modern hardened sites may use CSP only — check for both.

---

## Misconfiguration Pattern Summary

| Misconfiguration | Apache | Python HTTP | Node.js | Nginx |
|---|---|---|---|---|
| Version in headers | Yes | Yes | Partial | Yes |
| Directory listing | `/files/` | Root path | N/A | `/files/` |
| Status/debug endpoint | `/server-status` | N/A | `/api/debug/env`, `/api/routes` | `/nginx_status` |
| Sensitive files | `backup.bak` | `.env`, `backup.zip` | `config.js` | `server-config.txt` |
| Missing security headers | All | All | All | All |

---

## Automated Scanning with Nikto

```bash
# Basic scan
nikto -h http://TARGET:80 -nointeractive

# Targeted scan types (concatenate numbers, no commas)
nikto -h http://TARGET -Tuning 123

# Common findings Nikto surfaces on Apache:
# - /server-status exposure
# - Directory indexing on /files/
# - Missing X-Frame-Options
# - ETag inode leakage
# - backup.bak in document root
```

**Nikto is noisy** — generates lots of traffic, easy to detect in logs. Appropriate for authorized testing only.

---

## Engagement Workflow

```bash
# 1. Fingerprint all ports
for port in 80 8000 3000 8080 443 8443; do
    echo "=== Port $port ==="
    curl -sI http://TARGET:$port/ 2>/dev/null | grep -iE "server|x-powered-by"
done

# 2. Check error pages for version leakage
curl -s http://TARGET:PORT/nonexistent

# 3. Python HTTP server — always try these
curl -s http://TARGET:8000/
curl -s http://TARGET:8000/.env

# 4. Apache — check status and listings
curl -s http://TARGET:80/server-status
curl -s http://TARGET:80/files/

# 5. Node.js — check debug endpoints
curl -s http://TARGET:3000/api/routes
curl -s http://TARGET:3000/api/debug/env

# 6. Nginx — check status and listings  
curl -s http://TARGET:8080/nginx_status
curl -s http://TARGET:8080/files/

# 7. Security header audit
for port in 80 8000 3000 8080; do
    echo "=== Port $port ==="
    curl -sI http://TARGET:$port/ | grep -iE "x-frame|x-content|csp|hsts|referrer" || echo "(none)"
done

# 8. Gobuster for unlinked files
gobuster dir -u http://TARGET:80 \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x bak,txt,html -t 20

# 9. Nikto
nikto -h http://TARGET:80 -nointeractive
```
