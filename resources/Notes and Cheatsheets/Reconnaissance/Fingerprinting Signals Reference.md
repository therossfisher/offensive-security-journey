# Web Stack Fingerprinting — Signals and Examples

**Purpose:** Quick reference for identifying web stacks from HTTP responses. Each signal includes where to find it and what command surfaces it.

---

## The Problem with curl -I

`curl -I` sends a HEAD request and only shows response headers. Many signals live in the **response body** or only appear on **specific paths**. You need multiple commands to see the full picture.

```bash
curl -I http://TARGET:PORT/          # Headers only — misses body signals
curl -s http://TARGET:PORT/          # Full response body
curl -v http://TARGET:PORT/          # Headers + body + request details
curl http://TARGET:PORT/nonexistent  # Error page fingerprinting
curl -c cookies.txt http://TARGET/   # Capture cookies
```

---

## Express.js / MERN Stack

### Signal 1 — X-Powered-By Header
**Where:** Response headers  
**Command:** `curl -I http://TARGET:3000/`

```
HTTP/1.1 200 OK
X-Powered-By: Express          ← RIGHT HERE
Content-Type: text/html
```

### Signal 2 — connect.sid Cookie
**Where:** Response headers, but only on full GET requests that create a session  
**Command:** `curl -c cookies.txt http://TARGET:3000/`

```
Set-Cookie: connect.sid=s%3A2PyC5xblQ3G0ERkE60uOUddRtPs2jacn.0gAB6ByfrNg3b48tDXARTEBQG0pLlKkBofAsa69W%2FY0; Path=/; HttpOnly
```

Note: `-I` often misses this because HEAD requests don't always create sessions.

### Signal 3 — Cannot GET Error Format
**Where:** Response body of any nonexistent path  
**Command:** `curl http://TARGET:3000/nonexistent`

```html
<!DOCTYPE html>
<html lang="en">
<head><title>Error</title></head>
<body>
<pre>Cannot GET /nonexistent</pre>    ← Plain text, uniquely Express
</body>
</html>
```

Compare to other frameworks:
- Django → styled HTML error page
- Apache → styled 404 with version in footer
- Next.js → styled HTML 404 page

### Signal 4 — React Root Element
**Where:** HTML body  
**Command:** `curl -s http://TARGET:3000/ | grep "root"`

```html
<div id="root"></div>    ← React mount point in the HTML body
```

---

## Next.js Stack

### Signal 1 — X-Powered-By Header
**Where:** Response headers  
**Command:** `curl -I http://TARGET:3001/`

```
HTTP/1.1 200 OK
X-Powered-By: Next.js              ← Framework confirmed
x-nextjs-cache: HIT                ← Production mode confirmed
x-nextjs-prerender: 1              ← App Router confirmed
x-nextjs-stale-time: 4294967294
```

The three `x-nextjs-*` headers together confirm production build mode — required for CVE-2025-29927 to apply.

### Signal 2 — window.__next_f in Page Source
**Where:** HTML body, inside a script tag  
**Command:** `curl -s http://TARGET:3001/ | grep "next_f"`

```html
<script>(self.__next_f=self.__next_f||[]).push([0])</script>
```

This is the **definitive App Router indicator**. It's the hydration array for React Server Component data. Does not appear in Pages Router or any other framework.

### Signal 3 — Static Asset Paths
**Where:** HTML body, in script src attributes  
**Command:** `curl -s http://TARGET:3001/ | grep "_next/static"`

```html
<script src="/_next/static/chunks/webpack-db0a529a99835594.js"></script>
<script src="/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js"></script>
<script src="/_next/static/chunks/main-app-4fbb4b1f318e39a0.js"></script>
```

### Signal 4 — HTTP 307 Redirect on Protected Routes
**Where:** Response headers when requesting a protected page  
**Command:** `curl -v http://TARGET:3001/dashboard`

```
< HTTP/1.1 307 Temporary Redirect
< Location: /login
```

307 = middleware redirected you. This is the target for CVE-2025-29927. Without authentication, you get bounced. With the bypass header, you get through.

---

## Django Stack

### Signal 1 — Server Header
**Where:** Response headers  
**Command:** `curl -I http://TARGET:8000/`

```
Server: WSGIServer/0.2 CPython/3.10.12    ← Django-specific server banner
```

No other framework sends this. The Python version is also revealed.

### Signal 2 — csrftoken Cookie
**Where:** Response headers  
**Command:** `curl -c cookies.txt http://TARGET:8000/ && cat cookies.txt`

```
csrftoken    9vMaeHlURA0uOYnP9qB2BrDNTvNPoD0JPyecxWNxV7aohswgtAtBvwbLWaOTYIF7
```

### Signal 3 — Security Header Combination
**Where:** Response headers — all three together = Django SecurityMiddleware  
**Command:** `curl -I http://TARGET:8000/`

```
X-Frame-Options: DENY              ← Django default
X-Content-Type-Options: nosniff   ← Django default
Referrer-Policy: same-origin       ← Django default
```

No other framework applies all three by default. Seeing all three together is a near-certain Django fingerprint even if the Server header is stripped.

### Signal 4 — csrfmiddlewaretoken Hidden Field
**Where:** HTML body of any POST form — the most reliable Django fingerprint  
**Command:** `curl -s http://TARGET:8000/admin/login/ | grep csrf`

```html
<input type="hidden" name="csrfmiddlewaretoken" 
       value="w4VrwSsqEYpBZL4ROD1c4CgYbqw0zjZZeiXQVYGmkUjVDIce9k6wq7XvaORkbAkL">
```

Django's CsrfViewMiddleware injects this into every POST form automatically. You will not find this in Express, Rails, or any Next.js application.

---

## Apache / LAMP Stack

### Signal 1 — Server Header
**Where:** Response headers  
**Command:** `curl -I http://TARGET:8080/`

```
Server: Apache/2.4.49 (Unix)    ← Exact version = direct CVE mapping
```

Apache 2.4.49 → CVE-2021-41773 immediately.

### Signal 2 — Version in 404 Footer
**Where:** HTML body of error pages  
**Command:** `curl http://TARGET:8080/nonexistent`

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head><title>404 Not Found</title></head>
<body><h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.49 (Unix) Server at TARGET Port 8080</address>
</body></html>
```

Useful when a reverse proxy strips the Server header but error pages still leak the version.

### Signal 3 — /cgi-bin/ Response Code
**Where:** HTTP response code for that specific path  
**Command:** `curl -v http://TARGET:8080/cgi-bin/`

```
403 Forbidden    → Directory exists, mod_cgi enabled → CVE-2021-41773 applies
404 Not Found    → Directory doesn't exist → CVE doesn't apply
```

The difference between 403 and 404 here is the difference between exploitable and not exploitable.

---

## Full Fingerprinting Command Sequence

Run these in order against any new target:

```bash
TARGET="10.10.10.5"
PORT="8080"

# 1. Headers
curl -I http://$TARGET:$PORT/

# 2. Error format fingerprint
curl http://$TARGET:$PORT/nonexistent

# 3. Full page body
curl -s http://$TARGET:$PORT/

# 4. Save cookies
curl -c /tmp/cookies.txt http://$TARGET:$PORT/

# 5. Grep body for stack signals
curl -s http://$TARGET:$PORT/ | grep -i "next_f\|csrf\|root\|static\|powered"

# 6. Check common paths
curl -v http://$TARGET:$PORT/admin/
curl http://$TARGET:$PORT/cgi-bin/
curl http://$TARGET:$PORT/.env
curl http://$TARGET:$PORT/robots.txt

# 7. Run Nikto
nikto -h http://$TARGET:$PORT/

# 8. Match version to CVE
searchsploit apache 2.4.49
searchsploit django 3.2
```

---

## Signal Summary Table

| Signal | Stack | Where | Command |
|---|---|---|---|
| `X-Powered-By: Express` | Express/MERN | Headers | `curl -I` |
| `connect.sid` cookie | Express/MERN | Headers | `curl -c cookies.txt` |
| `Cannot GET /path` | Express/MERN | Body | `curl /nonexistent` |
| `<div id="root">` | React/MERN | Body | `curl -s /` |
| `X-Powered-By: Next.js` | Next.js | Headers | `curl -I` |
| `window.__next_f` | Next.js App Router | Body | `curl -s / \| grep next_f` |
| `/_next/static/chunks/` | Next.js | Body | `curl -s / \| grep _next` |
| `x-nextjs-cache` | Next.js production | Headers | `curl -I` |
| `HTTP 307 to /login` | Next.js protected route | Headers | `curl -v /dashboard` |
| `WSGIServer/0.2 CPython` | Django | Headers | `curl -I` |
| `csrftoken` cookie | Django | Headers | `curl -c cookies.txt` |
| `X-Frame-Options: DENY` | Django | Headers | `curl -I` |
| `csrfmiddlewaretoken` | Django | Body | `curl -s /admin/login/` |
| `Server: Apache/X.X.XX` | Apache | Headers | `curl -I` |
| Apache version in footer | Apache | Body | `curl /nonexistent` |
| `/cgi-bin/` returns 403 | Apache + mod_cgi | Status code | `curl /cgi-bin/` |
