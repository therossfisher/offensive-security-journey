# Cross-Site Request Forgery (CSRF)

**Category:** Web Vulnerabilities  
**OWASP:** A01 Broken Access Control (adjacent)  
**Impact:** Forces authenticated users to perform unintended actions without their knowledge

---

## What Is CSRF

CSRF abuses the trust relationship between a browser and a web application. When a user is logged into a site, their browser stores a session cookie. The browser automatically includes that cookie with every request to that domain — regardless of where the request originated.

An attacker exploits this by tricking an authenticated victim into visiting a malicious page that silently sends a forged request to the target application. From the server's perspective the request looks legitimate because it contains the victim's valid session cookie.

**The attacker never needs the victim's credentials.** They just need the victim's browser to send the request while the victim is already logged in.

---

## Three Conditions Required

All three must be true for a CSRF attack to work:

1. **Victim is authenticated** — active session with cookie stored in browser
2. **State-changing action exists** — endpoint that modifies data, not just reads it
3. **No origin verification** — application doesn't confirm the request came from a trusted source

---

## Common Vulnerable Features

Any feature that changes data is a potential target:

- Email address changes
- Password changes
- Account settings updates
- Role or permission changes
- Financial transactions
- Profile updates
- Adding or removing trusted devices

**Read-only actions are not CSRF targets** — CSRF only matters when the request changes something.

---

## How It Works — Step by Step

```
1. Victim logs into legitimate application
   → Browser stores session cookie

2. Victim visits attacker's malicious page
   → Could be a link in email, chat, social media, any site

3. Malicious page triggers a request to the target application
   → Hidden form, image tag, JavaScript redirect, etc.

4. Browser automatically includes session cookie
   → Server sees valid authenticated request

5. Server processes the action
   → Email changed, role modified, money transferred, etc.

6. Victim has no idea this happened
```

---

## Attack Techniques

### Hidden Auto-Submitting Form

Most common technique. Page loads and immediately submits a form without any user interaction.

```html
<html>
<body>
<form action="http://TARGET/update_email.php" method="POST" id="attack">
  <input type="hidden" name="email" value="attacker@evil.com">
</form>
<script>
  document.getElementById("attack").submit();
  setTimeout(function() {
    window.location.href = "http://TARGET/settings.php";
  }, 1000);
</script>
</body>
</html>
```

**What happens:** Form submits automatically on page load. setTimeout redirects victim to legitimate page after 1 second so they don't notice anything unusual.

### Image Tag (GET Request)

For endpoints that accept GET requests. The image src triggers the request when the page loads.

```html
<img src="http://TARGET/delete_account?confirm=yes" width="0" height="0">
```

Zero size image — victim sees nothing. Browser fetches the URL and the action executes.

### Onmouseover Event (GET Request)

Requires user interaction but more subtle. Action triggers when victim moves mouse over an element.

```html
<img src="http://TARGET/banner.png"
  onmouseover="window.location='http://TARGET/update_role.php?role=staff&csrf_token=TOKEN'"
  width="400">
```

---

## Bypassing Weak CSRF Tokens

Some applications implement CSRF tokens but generate them poorly. If the token is predictable or reversible it can be defeated.

**Common weak token patterns:**

| Pattern | Example | How to Reverse |
|---|---|---|
| Base64 encoded role/username | `YWRtaW4=` → `admin` | Decode with CyberChef or Burp Decoder |
| MD5 of username | `5f4dcc3b5aa765d61d8327deb882cf99` | Recognizable hash format |
| Sequential numbers | `token=1042` | Increment and try |
| Timestamp based | `token=1718300000` | Predictable from Unix time |

**Testing approach:**
1. Decode/reverse the token to understand the generation method
2. Reproduce the token using the same method
3. Include the reproduced token in your malicious request

**A properly implemented CSRF token must be:**
- Cryptographically random
- Unique per session or per request
- Server-side validated
- Not predictable from any user data

---

## Setting Up a Malicious Server for Testing

When testing CSRF on an external target you need a web server to host the malicious page.

```bash
# Create the malicious page
mkdir -p ~/csrf-test
nano ~/csrf-test/attack.html
# paste malicious HTML

# Serve it
cd ~/csrf-test
sudo python3 -m http.server 81
```

Page accessible at: `http://YOUR_IP:81/attack.html`

**Important:** The victim must visit your malicious URL in the **same browser and same session** where they are authenticated to the target. Opening in a different browser or incognito window won't work — there's no session cookie.

**Clear browser cache** between tests to avoid stale session state interfering with results.

---

## Identifying CSRF Vulnerabilities — Testing Methodology

**Step 1 — Map state-changing endpoints:**
Browse the application with Burp proxy running. Identify every request that modifies data. Focus on POST requests but don't ignore GET requests.

**Step 2 — Inspect for CSRF tokens:**
Check every state-changing request for:
- Hidden form fields (common names: `csrf_token`, `_token`, `authenticity_token`, `nonce`)
- Custom headers (`X-CSRF-Token`, `X-Requested-With`)
- Referrer header validation

**Step 3 — Analyze the token:**
If a token exists:
- Is it present in every sensitive request?
- Does it change with each request or session?
- Can it be decoded or reversed?
- What happens if you remove it entirely?
- What happens if you use a token from a different session?

**Step 4 — Reproduce outside the application:**
Copy the request and try to reproduce it from an external HTML page. If it succeeds without a valid token the endpoint is vulnerable.

**Step 5 — Check cookie behavior:**
Does the application rely solely on session cookies? SameSite cookie attributes can provide CSRF protection — check the Set-Cookie response header.

---

## Defenses (Developer Perspective)

**CSRF Tokens (correct implementation):**
- Generate cryptographically random token per session
- Embed in every state-changing form as hidden field
- Validate server-side on every POST request
- Reject requests with missing or invalid tokens

**SameSite Cookie Attribute:**
```
Set-Cookie: session=abc123; SameSite=Strict
Set-Cookie: session=abc123; SameSite=Lax
```
- `Strict` — cookie never sent with cross-site requests
- `Lax` — cookie sent with top-level navigations (GET) but not cross-site POST
- `None` — cookie sent with all requests (requires Secure flag)

**Origin/Referrer Header Validation:**
Check that the request originated from your own domain. Can be bypassed if misconfigured.

**Custom Request Headers:**
Require a custom header (like `X-Requested-With: XMLHttpRequest`) — cross-site forms can't set custom headers without CORS preflight.

---

## Common Misconceptions

**"POST requests are safe from CSRF"** — False. POST forms can be forged just as easily as GET requests using a hidden auto-submitting form.

**"HTTPS prevents CSRF"** — False. HTTPS encrypts the connection but doesn't prevent forged requests. The cookie is still sent automatically over HTTPS.

**"Requiring a password confirmation protects against CSRF"** — True only if the attacker doesn't know the password. Re-authentication is an effective CSRF defense for high-risk actions.

**"My login page needs CSRF protection"** — Usually not. CSRF requires the victim to already be authenticated. Login forms are more vulnerable to credential stuffing and brute force.

---

## Pentest Notes

- CSRF is about **what** the victim does, not **who** the attacker is
- The attacker never receives the response — they only trigger the action
- Social engineering is required to get the victim to visit the malicious page
- Most modern frameworks (Django, Rails, Laravel, Spring) include CSRF protection by default — look for places where developers disabled it
- Mobile APIs often lack CSRF protection because they use token-based auth instead of cookies — different attack surface
- SameSite=Lax is now the default in Chrome — reduces but doesn't eliminate CSRF risk
- Always test CSRF in the same browser session as the authenticated victim — different browser = no session cookie = no attack
