# Nikto Cheat Sheet

Nikto — automated web server scanner. Quick first pass for stack fingerprinting, missing security headers, misconfigurations, and known vulnerabilities. Pre-installed on Kali.

---

## Basic Syntax

```bash
nikto -h TARGET
nikto -h http://TARGET:PORT/
nikto -h https://TARGET
```

---

## Common Examples

```bash
# Basic scan
nikto -h http://TARGET

# Specific port
nikto -h http://TARGET:3000
nikto -h http://TARGET:8080

# HTTPS (skip cert verification)
nikto -h https://TARGET -ssl

# Save output to file
nikto -h http://TARGET -o results.txt
nikto -h http://TARGET -o results.html -Format html

# Scan multiple hosts from file
nikto -h hosts.txt

# Specific scan tuning (see tuning options below)
nikto -h http://TARGET -Tuning 1

# Set custom User-Agent
nikto -h http://TARGET -useragent "Mozilla/5.0"

# Use a proxy (route through Caido/Burp)
nikto -h http://TARGET -useproxy http://127.0.0.1:8080

# Scan with authentication
nikto -h http://TARGET -id admin:password

# Increase timeout
nikto -h http://TARGET -timeout 10

# Verbose output
nikto -h http://TARGET -Display V
```

---

## Key Flags

| Flag | Description |
|---|---|
| `-h` | Target host (required) |
| `-p` | Port (or specify in URL) |
| `-ssl` | Force SSL/HTTPS |
| `-o` | Output file |
| `-Format` | Output format: txt, html, csv, xml, json |
| `-Tuning` | Scan type filter (see below) |
| `-useproxy` | Route through proxy |
| `-id` | Basic auth credentials (user:pass) |
| `-useragent` | Custom User-Agent string |
| `-timeout` | Request timeout in seconds |
| `-Display` | Control what's shown (V=verbose, 1=redirects) |
| `-nossl` | Disable SSL |
| `-nolookup` | Skip hostname lookup |
| `-evasion` | IDS evasion techniques |

---

## Tuning Options

Control which tests Nikto runs. Combine numbers for multiple test types.

| Number | Test Type |
|---|---|
| 1 | Interesting files / seen in logs |
| 2 | Misconfiguration / default files |
| 3 | Information disclosure |
| 4 | Injection (XSS/Script/HTML) |
| 5 | Remote file retrieval (inside web root) |
| 6 | Denial of service |
| 7 | Remote file retrieval (server wide) |
| 8 | Command execution / remote shell |
| 9 | SQL injection |
| 0 | File upload |
| a | Authentication bypass |
| b | Software identification |
| c | Remote source inclusion |

```bash
# Only test for SQL injection and info disclosure
nikto -h http://TARGET -Tuning 93

# Only software identification (fastest for fingerprinting)
nikto -h http://TARGET -Tuning b
```

---

## What Nikto Finds

### Stack Fingerprinting
```
+ Retrieved x-powered-by header: Express        → MERN/Express
+ Retrieved x-powered-by header: Next.js        → Next.js App Router
+ Server: WSGIServer/0.2 CPython/3.10.12        → Django
+ Server: Apache/2.4.49 (Unix)                  → Apache (check CVE-2021-41773)
+ Server: nginx/1.18.0 (Ubuntu)                 → Nginx
+ Server: Microsoft-IIS/10.0                    → IIS/Windows
```

### Security Header Findings
```
+ The anti-clickjacking X-Frame-Options header is not present
+ The X-Content-Type-Options header is not set
+ No CGI Directories found
+ Cookie connect.sid created without the httponly flag
+ Cookie session created without the secure flag
```

### Dangerous Configurations
```
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE
+ OSVDB-877: HTTP TRACE method is active (XST vulnerability)
+ Server leaks inodes via ETags
+ /admin/ - Admin login page found
+ /.env - Environment file exposed
+ /backup.zip - Backup file found
```

---

## Reading Nikto Output

```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.10.10.5
+ Target Hostname:    10.10.10.5
+ Target Port:        8080
---------------------------------------------------------------------------
+ Server: Apache/2.4.49 (Unix)          ← Stack identified, check for CVEs
+ The anti-clickjacking X-Frame-Options header is not present.   ← Missing header
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS, TRACE          ← TRACE = XST risk
+ OSVDB-877: HTTP TRACE method is active                         ← Confirmed finding
+ /cgi-bin/: Directory found            ← mod_cgi present
---------------------------------------------------------------------------
+ 6544 items checked: 0 error(s) and 5 item(s) reported
```

**Each finding line starts with `+`**. The number after OSVDB is the vulnerability database ID — searchable at osvdb.org (archived) or vulners.com.

---

## What Nikto Misses

Nikto is a first pass scanner. It does NOT find:

- Application-level injection (prototype pollution, custom SQL injection)
- Logic vulnerabilities
- IDOR (Insecure Direct Object Reference)
- Authentication bypass via custom headers (CVE-2025-29927)
- Business logic flaws
- Anything that requires authentication to reach

After Nikto, manual testing takes over for application-layer vulnerabilities.

---

## Fingerprinting Workflow with Nikto

```bash
# Step 1 — Quick header check
curl -I http://TARGET:PORT/

# Step 2 — Run Nikto for automated first pass
nikto -h http://TARGET:PORT/

# Step 3 — Note the stack from Nikto output
# Step 4 — Search for CVEs based on version
searchsploit apache 2.4.49
searchsploit django 3.2

# Step 5 — Manual testing for application-layer vulnerabilities
```

---

## Nikto + Proxy (Route Through Caido)

Send Nikto traffic through Caido so you can see and replay the requests:

```bash
nikto -h http://TARGET -useproxy http://127.0.0.1:8080
```

This lets you see exactly what Nikto is sending and capture interesting responses for manual follow-up.

---

## Common Findings and What to Do With Them

| Nikto Finding | What It Means | Next Step |
|---|---|---|
| `Server: Apache/2.4.49` | Exact CVE match | Run CVE-2021-41773 exploit |
| `X-Powered-By: Express` | Node.js backend | Test for prototype pollution |
| `X-Powered-By: Next.js` | Check for App Router | Test CVE-2025-29927 |
| `TRACE method enabled` | XST possible | Test with `curl -X TRACE` |
| `Missing X-Frame-Options` | Clickjacking possible | Document as finding |
| `Cookie without HttpOnly` | XSS can steal cookie | Note for XSS testing |
| `Cookie without Secure` | Cookie sent over HTTP | Document as finding |
| `/.env found` | Credentials exposed | Immediately read the file |
| `/admin/ found` | Admin panel exists | Test default credentials |
| `/backup.zip found` | Source code exposed | Download and analyze |
| `ETags leak inodes` | Info disclosure | Document as finding |

---

## Notes
- Nikto is noisy — it makes thousands of requests and will show up in logs
- Not suitable for stealth engagements
- Always run with permission — unauthorized scanning is illegal
- Use `-Tuning b` for fastest stack identification when that's all you need
- Combine with `searchsploit` to go from version number to working exploit
