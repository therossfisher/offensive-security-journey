# Burp Suite — Repeater

Repeater captures requests and lets you modify and resend them manually as many times as needed. Essential for manual vulnerability testing, payload crafting, and endpoint analysis.

---

## What It's For

- Manually testing endpoints with different input values
- Crafting payloads through trial and error (SQLi, XSS, etc.)
- Analyzing how the server responds to specific inputs
- Testing the same request repeatedly with minor changes
- Building requests from scratch (like curl but with a GUI)

---

## Interface Layout

| Section | Location | Purpose |
|---|---|---|
| Request List | Top left | All requests sent to Repeater — manage multiple simultaneously |
| Request Controls | Below request list | Send, cancel, navigate history |
| Request View | Left main panel | Edit the request here before sending |
| Response View | Right main panel | Server response shown here after sending |
| Layout Options | Top right | Switch between horizontal, vertical, or tabbed layout |
| Inspector | Far right | Visual breakdown of request/response components |
| Target | Above Inspector | IP/domain the request goes to — auto-populated from Proxy |

---

## Sending Requests to Repeater

From Proxy or anywhere else in Burp:
- Right-click request → Send to Repeater
- Keyboard shortcut: **Ctrl + R**

---

## Response View Options

Four display modes above the response box:

| Mode | Use |
|---|---|
| **Pretty** | Default — slight formatting for readability |
| **Raw** | Unmodified server response |
| **Hex** | Byte-level view — useful for binary files |
| **Render** | Visual browser-like rendering of the page |

**Show non-printable characters** button (`\n`) — reveals characters like `\r\n` (carriage return + newline) at end of HTTP header lines.

---

## Inspector

Provides a structured breakdown of request and response components. Easier than editing raw text for specific changes.

**Modifiable sections:**
- Request Attributes — URL path, HTTP method (GET/POST/etc.), protocol (HTTP/1 vs HTTP/2)
- Request Query Parameters — data passed in URL (`?key=value`)
- Request Body Parameters — POST data
- Request Cookies — cookies sent with the request
- Request Headers — add, remove, or edit headers

**Read-only:**
- Response Headers — shown after sending, cannot be modified

---

## Practical Uses

**Adding a custom header to test access control:**
```
FlagAuthorised: True
```
Add in Inspector → Request Headers → type header name and value

**Testing for SQL injection:**
Add `'` to the end of a parameter and observe if the server returns an error — confirms SQLi vulnerability exists.

**Union-based SQLi example workflow in Repeater:**
```
/about/2'                          -- break the query, confirm error
/about/0 UNION ALL SELECT 1,2,3,4,5  -- find column count
/about/0 UNION ALL SELECT group_concat(column_name),null,null,null,null FROM information_schema.columns WHERE table_name="TARGET_TABLE"
/about/0 UNION ALL SELECT notes,null,null,null,null FROM TARGET_TABLE WHERE id=1
```

**XSS bypass via Burp (bypassing client-side filters):**
1. Submit legitimate data through the form
2. Intercept the request in Proxy
3. Forward to Repeater
4. Change the email/input field to XSS payload
5. URL encode the payload: select it → Ctrl + U
6. Forward — server-side filter not present, payload executes

**Testing numeric endpoints for poor validation:**
- Capture request to `/products/3`
- Send to Repeater
- Change `3` to extreme values — very large numbers, negative numbers, decimals, strings
- 500 error = server not validating input properly

---

## History Navigation

Buttons to the right of the Send button let you navigate forward and backward through all modifications made to the request in the current session.

---

## URL Encoding in Repeater

When sending payloads with special characters:
- Select the text you want to encode
- **Ctrl + U** — URL encodes selected text
- Prevents special characters from being misinterpreted

---

## Key Difference from Proxy

Proxy intercepts traffic passively. Repeater lets you actively craft and resend requests as many times as needed with full control over every part of the request.
