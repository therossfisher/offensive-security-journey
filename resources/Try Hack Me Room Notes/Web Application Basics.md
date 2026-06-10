# Web Application Basics

**THM Room:** Web Application Basics  
**Path:** Jr Penetration Tester / Web Application Security  
**Completed:** June 2026

---

## Web Application Components

| Component | Role | Analogy |
|---|---|---|
| **HTML** | Structure and content | DNA — instructions for how things are built |
| **CSS** | Styling and appearance | DNA describing color, shape, size |
| **JavaScript** | Dynamic behavior and logic | Brain — makes decisions based on interactions |
| **Web Server** | Hosts and delivers content | Planet infrastructure |
| **Database** | Stores, modifies, retrieves data | Library/filing cabinet |
| **WAF** | Filters malicious requests | Planet atmosphere filtering UV rays |

**Front End** — what users see and interact with (HTML, CSS, JS)  
**Back End** — everything powering it underneath (web server, database, application logic)

---

## URL Anatomy

```
https://user:password@tryhackme.com:443/path/to/page?search=query#section
  |      |              |              |   |              |           |
scheme  user           host           port path        query       fragment
```

| Component | Description | Security Note |
|---|---|---|
| **Scheme** | Protocol (HTTP/HTTPS) | Always prefer HTTPS |
| **User** | Login credentials in URL | Rare and insecure — exposes creds |
| **Host/Domain** | Website address | Watch for typosquatting in phishing |
| **Port** | Service doorway (80=HTTP, 443=HTTPS) | Non-standard ports worth noting in recon |
| **Path** | Specific resource location | Validate to prevent unauthorized access |
| **Query String** | Starts with `?` — passes parameters | User-modifiable — test for injection |
| **Fragment** | Starts with `#` — jumps to page section | User-modifiable — sanitize input |

**Typosquatting** — registering misspelled domain names to trick users (e.g. paypa1.com)

---

## HTTP Messages

Two types:
- **HTTP Request** — client sends to server to trigger action
- **HTTP Response** — server sends back after processing request

**Message structure:**
```
Start Line      (method/path/version OR status line)
Headers         (key: value pairs)
                (empty line separator)
Body            (optional — data being sent or received)
```

---

## HTTP Request Methods

| Method | Purpose | Security Note |
|---|---|---|
| **GET** | Fetch data, no changes | Don't put sensitive data in GET — visible in URL/logs |
| **POST** | Send data to create/update | Validate and sanitize input — SQLi, XSS risk |
| **PUT** | Replace/update resource | Verify authorization before accepting |
| **DELETE** | Remove resource | Restrict to authorized users only |
| **PATCH** | Partial update | Validate to avoid inconsistencies |
| **HEAD** | Like GET but headers only | Check metadata without downloading content |
| **OPTIONS** | List available methods | Often disabled in production — recon value |
| **TRACE** | Debug — shows allowed methods | Usually disabled — security risk |
| **CONNECT** | Create secure tunnel | Used for HTTPS proxying |

---

## HTTP Versions

| Version | Year | Key Features |
|---|---|---|
| HTTP/0.9 | 1991 | GET only |
| HTTP/1.0 | 1996 | Headers, content types |
| HTTP/1.1 | 1997 | Persistent connections, chunked encoding — still widely used |
| HTTP/2 | 2015 | Multiplexing, header compression, faster |
| HTTP/3 | 2022 | Built on QUIC (UDP) — faster, more secure |

---

## Request Headers

| Header | Example | Purpose |
|---|---|---|
| `Host` | `Host: tryhackme.com` | Specifies target web server |
| `User-Agent` | `User-Agent: Mozilla/5.0` | Identifies browser/client |
| `Referer` | `Referer: https://google.com` | Where request came from |
| `Cookie` | `Cookie: session=abc123` | Stored data sent back to server |
| `Content-Type` | `Content-Type: application/json` | Format of request body data |

---

## Request Body Formats

**URL Encoded** (`application/x-www-form-urlencoded`)
```
name=Ross&age=37&country=US
```
Default for HTML form submissions. Key=value pairs separated by `&`.

**Form Data** (`multipart/form-data`)
Used for file uploads. Data blocks separated by boundary strings. Supports binary data.

**JSON** (`application/json`)
```json
{
    "name": "Ross",
    "age": 37,
    "country": "US"
}
```
Most common for APIs.

**XML** (`application/xml`)
```xml
<user>
    <name>Ross</name>
    <age>37</age>
</user>
```
Older format, still seen in enterprise/SOAP APIs.

---

## HTTP Response Status Codes

| Range | Category | Meaning |
|---|---|---|
| 100-199 | Informational | Server received request, keep going |
| 200-299 | Success | Request processed successfully |
| 300-399 | Redirection | Resource has moved |
| 400-499 | Client Error | Problem with the request |
| 500-599 | Server Error | Problem on the server side |

**Common codes:**

| Code | Meaning |
|---|---|
| 200 | OK — success |
| 301 | Moved Permanently — use new URL |
| 404 | Not Found — resource doesn't exist |
| 500 | Internal Server Error — server-side issue |

---

## Response Headers

| Header | Example | Purpose |
|---|---|---|
| `Date` | `Date: Fri, 07 Jun 2026 10:00:00 GMT` | When response was generated |
| `Content-Type` | `Content-Type: text/html; charset=utf-8` | Type of content being returned |
| `Server` | `Server: nginx` | Web server software — obscure this in production |
| `Set-Cookie` | `Set-Cookie: sessionId=abc123` | Sets cookies on client |
| `Cache-Control` | `Cache-Control: max-age=600` | How long client can cache response |
| `Location` | `Location: /index.html` | Where to redirect (3xx responses) |

**Cookie security flags:**
- `HttpOnly` — prevents JavaScript from accessing the cookie (XSS protection)
- `Secure` — cookie only sent over HTTPS

---

## Security Headers

### Content-Security-Policy (CSP)
Defines trusted sources for content — mitigates XSS.
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'
```

| Directive     | Purpose                          |
| ------------- | -------------------------------- |
| `default-src` | Default policy for all content   |
| `script-src`  | Where scripts can be loaded from |
| `style-src`   | Where CSS can be loaded from     |
| `'self'`      | Same domain as current site      |


### Strict-Transport-Security (HSTS)
Forces HTTPS connections — prevents SSL stripping.
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

| Directive | Purpose |
|---|---|
| `max-age` | How long to enforce (seconds) |
| `includeSubDomains` | Apply to all subdomains |
| `preload` | Include in browser preload lists |

### X-Content-Type-Options
Prevents browsers guessing MIME types — mitigates content sniffing attacks.
```
X-Content-Type-Options: nosniff
```

### Referrer-Policy
Controls how much referrer information is shared.
```
Referrer-Policy: no-referrer            # Send nothing
Referrer-Policy: same-origin            # Send only on same-origin requests
Referrer-Policy: strict-origin          # Send origin only, same protocol
Referrer-Policy: strict-origin-when-cross-origin  # Full URL same-origin, origin only cross-origin
```

---

## Pentest Relevance

- **Query strings and fragments** — user-modifiable, always test for injection
- **OPTIONS method** — reveals what methods are allowed, useful in recon
- **Server header** — leaks web server software and version
- **Location header** — if user-controllable, test for open redirect
- **Cookies without HttpOnly/Secure** — XSS and interception risk
- **Missing CSP** — XSS attack surface
- **Missing HSTS** — SSL stripping possible
- **HTTP methods enabled unnecessarily** — PUT/DELETE can be dangerous if unrestricted

**Tool:** https://securityheaders.io — analyze any site's security headers instantly
