# Burp Suite — Intruder

Intruder automates sending modified requests with different payload values. Used for fuzzing endpoints, brute forcing credentials, and testing parameters at scale. Rate limited in Community edition — use ffuf or wfuzz for speed-sensitive tasks.

---

## What It's For

- Brute forcing login forms
- Fuzzing endpoints and parameters
- Credential stuffing attacks
- IDOR testing across a range of IDs
- Fuzzing for subdomains, directories, or hidden parameters
- Testing how an application handles unexpected input at scale

---

## Interface — Four Sub-tabs

| Tab | Purpose |
|---|---|
| **Positions** | Define where payloads get inserted in the request |
| **Payloads** | Configure what values get inserted |
| **Resource Pool** | Resource allocation — not useful in Community |
| **Settings** | Configure attack behavior, grep for responses, handle redirects |

---

## Positions Tab

Burp auto-highlights likely injection points in green, wrapped with `§` symbols.

**Buttons:**
- **Add §** — manually define a new position by highlighting text then clicking
- **Clear §** — remove all defined positions
- **Auto §** — auto-detect positions (restores defaults)

The `§` symbol marks the start and end of each payload position.

---

## Attack Types

### Sniper
- Default and most common
- One payload set, one position at a time
- Cycles through all payloads for each position sequentially
- Total requests = number of payloads × number of positions
- **Best for:** Single parameter fuzzing, password brute force, API endpoint fuzzing

### Battering Ram
- One payload set
- Same payload inserted into ALL positions simultaneously
- Total requests = number of payloads
- **Best for:** Testing race conditions, same value across multiple fields

### Pitchfork
- Multiple payload sets (one per position, max 20)
- Iterates all sets simultaneously — first payload from set 1 with first payload from set 2, etc.
- Stops when shortest list runs out
- Total requests = length of shortest payload list
- **Best for:** Credential stuffing where you have matched username:password pairs

### Cluster Bomb
- Multiple payload sets (one per position, max 20)
- Tests every possible combination of all payload sets
- Total requests = product of all list lengths
- **Best for:** Brute force where mapping between parameters is unknown (e.g., don't know which password belongs to which username)

---

## Payloads Tab — Four Sections

**Payload Sets:**
- Choose which position to configure
- Select payload type (Simple list, Numbers, Dates, Brute forcer, etc.)

**Payload Settings:**
- Add individual payloads manually
- Paste from clipboard
- Load from file (wordlist)
- Remove individual items or clear all

**Payload Processing:**
- Rules applied to each payload before sending
- Add prefix/suffix
- Match and replace
- Skip payloads matching a regex
- Capitalize, URL encode, etc.

**Payload Encoding:**
- URL encoding applied by default
- Can override which characters get encoded

---

## Common Payload Sources

```
/usr/share/wordlists/rockyou.txt              # passwords
/usr/share/seclists/Discovery/Web-Content/common.txt  # directories
/usr/share/seclists/Usernames/top-usernames-shortlist.txt  # usernames
```

For numeric fuzzing (IDOR testing):
- Payload type → Numbers
- Set range (e.g., 1 to 100, step 1)

---

## Identifying Successful Responses

Most attacks return the same status code for both success and failure. Sort results by:
- **Length** — successful responses are often shorter (redirect) or longer (content returned)
- **Status code** — look for 200 among 302 redirects, or 302 among 403/404
- **Response time** — timing attacks on blind injection

---

## Practical Examples

**Credential Stuffing with Pitchfork:**
1. Capture login POST request in Proxy
2. Send to Intruder (Ctrl + I)
3. Positions tab — clear all, manually select username and password fields
4. Set attack type to Pitchfork
5. Payloads tab — load usernames.txt into set 1, passwords.txt into set 2
6. Start Attack
7. Sort by Length to find successful login

**IDOR Fuzzing with Sniper:**
1. Capture request to `/support/ticket/3`
2. Send to Intruder
3. Positions tab — highlight the number `3`, click Add §
4. Attack type: Sniper
5. Payloads tab — Numbers, 1 to 100, step 1
6. Start Attack
7. Look for 200 responses among 302/403/404s

**Bypassing CSRF Token Protection with Macros:**

When a login form has a CSRF token that changes with every request:
1. Settings → Sessions → Macros → Add
2. Select a GET request to the login page
3. Name the macro, click OK
4. Sessions → Session Handling Rules → Add
5. Scope tab — Tools Scope: check only Intruder
6. URL Scope: use suite scope
7. Details tab → Rule Actions → Add → Run a Macro → select macro
8. Configure to update only `loginToken` parameter and `session` cookie
9. Run Intruder attack — macro fetches fresh token for every request

**Note:** All responses will be 302 if CSRF macro is working correctly. 403 errors mean the macro isn't working.

---

## Rate Limiting in Community

Intruder in Community edition is significantly throttled. For speed-sensitive attacks use:
```bash
# ffuf — faster directory and parameter fuzzing
ffuf -u http://TARGET/FUZZ -w WORDLIST

# wfuzz — flexible fuzzing
wfuzz -c -z file,WORDLIST http://TARGET/FUZZ

# hydra — faster brute force
hydra -L users.txt -P passwords.txt TARGET http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
```

Intruder is still worth learning for understanding the concepts and for low-volume manual testing.
