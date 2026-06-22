# JavaScript Essentials

JavaScript (JS) is a client-side scripting language that adds interactivity to HTML/CSS web pages. From a security perspective, JS is critical to understand because it runs in the user's browser, can be viewed and manipulated by anyone, and is a primary attack surface for web application vulnerabilities like XSS.

---

## Core Concepts

### Variables

Three ways to declare variables:

```javascript
var x = 5;      // function-scoped — older style, avoid in modern code
let y = 10;     // block-scoped — preferred for values that change
const z = 15;   // block-scoped — preferred for values that don't change
```

**Security relevance:** Developers sometimes hardcode sensitive values into variables (API keys, passwords, tokens). Since JS runs client-side, anyone can view it in the browser.

### Data Types

```javascript
let name = "Ross";        // string
let age = 30;             // number
let active = true;        // boolean
let nothing = null;       // null
let undef;                // undefined
let obj = {key: "value"}; // object
let arr = [1, 2, 3];      // array
```

### Functions

```javascript
function greet(name) {
    alert("Hello " + name);
}

greet("Ross"); // calling the function
```

### Loops

```javascript
// for loop — runs a set number of times
for (let i = 0; i < 10; i++) {
    console.log(i);
}

// while loop — runs while condition is true
while (condition) {
    // code
}

// do...while — runs at least once, then checks condition
do {
    // code
} while (condition);
```

### Conditional Statements

```javascript
if (age >= 18) {
    console.log("Adult");
} else {
    console.log("Minor");
}
```

---

## Integrating JS into HTML

### Internal JS
JS embedded directly in the HTML file inside `<script>` tags:

```html
<body>
    <p id="result"></p>
    <script>
        let x = 5;
        let y = 10;
        document.getElementById("result").innerHTML = x + y;
    </script>
</body>
```

### External JS
JS in a separate `.js` file, linked via the `src` attribute:

```html
<script src="script.js"></script>
```

**How to identify which type a site uses:** View page source → look for `<script>` tags. If the tag has a `src` attribute, it's external. If it contains code directly, it's internal.

**Security relevance:** External JS files can be swapped, tampered with, or replaced with malicious versions. Poorly vetted third-party libraries are a real attack vector.

---

## Dialogue Functions

JS has three built-in dialogue functions:

```javascript
alert("Message");              // displays message, OK button only
prompt("Enter your name:");    // asks for user input, returns the value
confirm("Are you sure?");      // returns true (OK) or false (Cancel)
```

### How attackers abuse dialogue functions

An attacker can send a malicious HTML file that spams alert boxes:

```javascript
for (let i = 0; i < 500; i++) {
    alert("Hacked");
}
```

More seriously — if user input from `prompt()` is passed directly into the DOM without sanitization, it becomes an XSS vector. This is the foundation of stored and reflected XSS attacks you'll see on HTB/THM web boxes.

---

## Control Flow Bypass (Security Perspective)

If authentication logic is implemented purely in client-side JS, it can be bypassed by reading and manipulating the source code directly in the browser.

**Example of bad practice — hardcoded credentials in JS:**
```javascript
function login(username, password) {
    if (username === "admin" && password === "ComplexPassword") {
        alert("Logged in!");
    }
}
```

Anyone can view page source, find the password, and log in. This is a real finding in web app pentests.

**How to find it:** Right-click → View Page Source → search for `password`, `pass`, `secret`, `key`, `token`, `auth`.

**Browser console bypass:** Open DevTools console (F12 or Ctrl+Shift+I) and call the login function directly or modify variables:
```javascript
// If the condition checks a variable, just set it directly
isLoggedIn = true;
```

---

## Minification and Obfuscation

**Minification** removes whitespace, comments, and shortens variable names to reduce file size. Code still functions identically.

**Obfuscation** makes code intentionally hard to read — renames variables to meaningless strings, adds dummy code, encodes strings.

```javascript
// Original
function hi() {
    alert("Welcome to THM");
}

// Obfuscated (same function, unreadable)
(function(_0x114713,_0x2246f2){var _0x51a830=_0x33bf ...
```

### Deobfuscating JS

Useful for CTFs and real web app pentests when you find minified/obfuscated client-side code:

- **Deobfuscate:** https://deobfuscate.io
- **Beautify/format:** https://beautifier.io
- **Browser DevTools:** Sources tab → click `{}` (pretty print) at bottom of file

**On HTB/THM web boxes:** Always check the page source and JS files for hardcoded credentials, API keys, hidden endpoints, or logic you can bypass. Minified files are worth deobfuscating.

---

## Developer Tools for Web App Pentesting

Open with F12 or Ctrl+Shift+I in Chrome/Firefox:

| Tab | What it shows |
|---|---|
| Console | Run JS directly, see errors and logs |
| Sources | All JS files loaded by the page — read and set breakpoints |
| Network | All HTTP requests/responses — useful for finding hidden API endpoints |
| Elements | Live DOM — modify HTML/CSS in real time |
| Application | Cookies, localStorage, sessionStorage |

**Console tricks:**
```javascript
// Read a variable value
console.log(variableName);

// Modify a variable
variableName = "newvalue";

// Call a function directly
login("admin", "password");

// Read cookies
document.cookie;

// Read localStorage
localStorage;
```

---

## Security Best Practices (and Why They Matter for Pentesting)

| Bad Practice | Why It's a Problem | What to Look For |
|---|---|---|
| Client-side validation only | Can be bypassed in browser console | Login forms, age checks, access controls |
| Hardcoded credentials/API keys | Visible to anyone viewing source | Search source for `password`, `key`, `token`, `secret` |
| Untrusted third-party libraries | Malicious library can compromise the whole site | Check `<script src="">` tags for external domains |
| No minification/obfuscation | Logic fully readable by attackers | Everything is visible in page source |

---

## JS Attack Surface — What to Check on Every Web Box

```
1. View page source (Ctrl+U) — look for credentials, endpoints, comments
2. Open DevTools Sources tab — find and read all JS files
3. Check for hardcoded secrets — search for: password, key, token, secret, api
4. Look for client-side auth logic — can it be bypassed in console?
5. Check external script sources — are they trusted domains?
6. Deobfuscate any minified JS — beautifier.io or DevTools pretty print
7. Check Network tab while interacting — what API calls are being made?
8. Read JS for hidden endpoints or functionality not visible in the UI
```

---

## Connection to Other Topics

- **XSS** — unsanitized user input passed into the DOM via JS → [[XSS]]
- **CSRF** — JS making unintended requests on behalf of authenticated users → [[CSRF]]
- **Burp Suite** — intercept and modify the HTTP requests JS makes → [[1 Overview]]
- **Prompt injection / AI security** — similar concept of injecting into input that gets processed → [[Prompt Engineering - General Notes]]

---

## Resources

- MDN JavaScript docs: https://developer.mozilla.org/en-US/docs/Web/JavaScript
- JS Beautifier: https://beautifier.io
- JS Deobfuscator: https://deobfuscate.io
- JS Obfuscator (for understanding what obfuscated code looks like): https://obfuscator.io
- PortSwigger Web Security Academy (XSS, client-side topics): https://portswigger.net/web-security
