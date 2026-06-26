# Burp Suite — Other Modules & Extensions

---

## Decoder

Transforms data between encoding formats and generates hash sums. Useful for understanding and manipulating encoded data during testing.

### Encoding/Decoding Options

| Format | Use |
|---|---|
| **Plain** | Raw text, no transformation |
| **URL** | `%2F` format — used in web requests. Special chars → `%` + hex ASCII code |
| **HTML** | `&amp;` format — used in HTML to prevent XSS rendering |
| **Base64** | Converts binary/text to ASCII-safe format — common in tokens, cookies, auth headers |
| **ASCII Hex** | Converts between ASCII text and hex representation |
| **Hex** | Numeric values only — converts to hexadecimal |
| **Octal** | Numeric values only — converts to base 8 |
| **Binary** | Numeric values only — converts to binary |
| **Gzip** | Compress/decompress gzip data |

### Smart Decode
Auto-detects encoding and recursively decodes until plaintext. Equivalent to CyberChef's "Magic" function. Not perfect but useful for unknown encoding.

### Hashing
Generate hashsums of data. Output is binary — apply ASCII Hex encoding afterward to get the readable hex string format.

Common algorithms: MD5, SHA-1, SHA-256, SHA-512, MD4, and many more.

**Note:** MD5 is deprecated — don't use it for anything security-sensitive. SHA-256 minimum for modern applications.

### Workflow Example — Multi-step Encoding
1. Enter text in input box
2. Select Encode as → Base64
3. Output appears — select it
4. Encode as → ASCII Hex
5. Output appears — select it
6. Encode as → Octal
7. Final encoded string produced

### Hex View
View and edit data byte by byte. Essential for binary files and non-ASCII data.

### Sending to Decoder
Right-click any data in Burp → Send to Decoder. Or paste directly into the input box.

---

## Comparer

Compare two pieces of data side by side at word or byte level. Highlights added, modified, and deleted content.

### When to Use
- Compare login responses of different lengths to identify successful authentication
- Compare two API responses to spot differences in returned data
- Identify what changes between authenticated and unauthenticated responses
- Find differences in responses when fuzzing parameters

### How to Use
1. Send two responses to Comparer (right-click → Send to Comparer)
2. Select both items in the left panel
3. Click Words or Bytes
4. Comparison window shows differences highlighted by color

### Comparison Window
- Color key at bottom left shows what each color means (modified, deleted, added)
- Sync views checkbox keeps both panels in the same format
- Can switch between text and hex view
- Window title shows total number of differences found

### Practical Example
Brute forcing a login — all responses return 302 redirect. Send two responses of different lengths to Comparer → identify which fields differ → determine why one indicates success.

---

## Sequencer

Analyzes the randomness (entropy) of tokens. Used to determine if session cookies, CSRF tokens, or other generated values are cryptographically secure.

### Why It Matters
If tokens are predictable, an attacker could:
- Forge session cookies and hijack accounts
- Predict CSRF tokens and bypass CSRF protection
- Predict password reset tokens

### Two Modes

**Live Capture:**
- Send a request that generates a token to Sequencer
- Sequencer sends the same request thousands of times automatically
- Collects and analyzes the generated tokens
- More accurate with more samples (10,000+ recommended)

**Manual Load:**
- Load a pre-generated list of tokens directly
- Less noisy (fewer requests to target)
- Requires having the token list already

### Live Capture Workflow
1. Capture request to login/token endpoint in Proxy
2. Right-click → Send to Sequencer
3. Token Location → select Form field or Cookie → choose the token parameter
4. Click Start live capture
5. Wait for ~10,000 tokens to collect
6. Click Pause → Analyze now
7. Review the entropy report

### Reading the Report
- **Overall result** — broad assessment of token security
- **Effective entropy** — bits of randomness. 117+ bits = strong. Lower = potentially predictable
- **Reliability/significance level** — confidence in the result. 1% significance = 99% confidence
- **Sample** — details about tokens analyzed

### Auto Analyze
Check "Auto analyze" to get periodic updates every 2,000 requests as sampling continues.

---

## Organizer

Stores read-only copies of HTTP requests for later review. Useful for keeping track of interesting findings during a pentest or bug bounty engagement.

### What It Stores
Each saved request includes:
- Request index number
- Timestamp
- Workflow status
- Source module (Proxy, Repeater, etc.)
- HTTP method
- Hostname
- URL path and query string
- Parameter count
- Response status code
- Response length
- Notes field

### How to Use
- Right-click any request → Send to Organizer
- Keyboard shortcut: **Ctrl + O**
- Click any item to view the request and response (read-only)
- Search within request/response using the search bar below

### Saved requests are read-only
You cannot modify them — they are snapshots at the time of capture. Send to Repeater if you want to modify and resend.

### Use Cases
- Flag interesting endpoints during enumeration
- Save requests you want to include in a report
- Track requests that need further investigation
- Maintain a log of significant findings across a long engagement

---

## Extensions

Burp Suite supports third-party and custom extensions that add functionality to the framework.

### BApp Store
Official extension marketplace built into Burp Suite.

To access: Extensions tab → BApp Store sub-tab

Extensions can:
- Add new tabs to the main interface
- Add options to right-click menus
- Extend existing module functionality
- Automate specific testing tasks

### Notable Free Extensions Worth Knowing
- **Logger++** — extends built-in logging with advanced filtering
- **Request Timer** — logs response time for each request (useful for timing attacks)
- **Autorize** — tests for authorization issues automatically
- **CSRF Scanner** — scans for CSRF vulnerabilities
- **Param Miner** — discovers hidden parameters

### Installing an Extension
1. Extensions tab → BApp Store
2. Search for extension name
3. Click the extension → Install
4. New tab or right-click options appear after installation

### Extension Languages
Extensions can be written in:
- **Java** — integrates natively, no additional setup
- **Python** — requires Jython (Java implementation of Python)
- **Ruby** — requires JRuby (Java implementation of Ruby)

### Setting Up Jython (for Python extensions)
1. Download Jython Standalone JAR from jython.org
2. Extensions tab → Extensions settings sub-tab
3. Python environment section → set path to Jython JAR file

### Extension Execution Order
Extensions are applied in **descending** order — top of list runs first. Order matters when multiple extensions modify the same requests. Reorder using Up/Down buttons in the Extensions interface.

### Extension Details Panel
Three sections for each selected extension:
- **Details** — name, version, description
- **Output** — any output generated during execution
- **Errors** — errors encountered during execution (useful for debugging)

### Custom Extension Development
Burp Suite exposes APIs accessible from the APIs sub-tab in Extensions. Beyond the scope of most pentest work but useful for automating repetitive testing tasks specific to a target.

Reference: PortSwigger's official extension development documentation.
