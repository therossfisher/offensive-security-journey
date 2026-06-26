# Burp Suite — Overview

Burp Suite is a Java-based web application penetration testing framework. It intercepts and manipulates all HTTP/HTTPS traffic between a browser and web server. Industry standard tool for manual web and mobile application security assessments including APIs.

---

## Editions

| Edition | Use Case | Cost |
|---|---|---|
| Community | Learning and manual testing — rate limited | Free |
| Professional | Full featured — automated scanner, unlimited fuzzing, project saving | Paid |
| Enterprise | Server-based continuous scanning (like Nessus for web apps) | Paid |

**Community limitations worth knowing:**
- Intruder is heavily rate limited — use ffuf or wfuzz for speed
- Cannot save projects between sessions
- No automated vulnerability scanner

---

## Core Modules

| Module | Purpose |
|---|---|
| **Proxy** | Intercepts and logs HTTP/HTTPS traffic between browser and server |
| **Repeater** | Capture, modify, and resend requests manually — great for manual testing |
| **Intruder** | Automated fuzzing and brute forcing — rate limited in Community |
| **Decoder** | Encode/decode/hash data (URL, Base64, HTML, Hex, etc.) |
| **Comparer** | Compare two responses word by word or byte by byte |
| **Sequencer** | Analyze randomness/entropy of tokens (session cookies, CSRF tokens) |
| **Organizer** | Store and annotate requests to revisit later |

---

## Navigation

**Keyboard shortcuts:**

| Shortcut | Tab |
|---|---|
| Ctrl + Shift + D | Dashboard |
| Ctrl + Shift + T | Target |
| Ctrl + Shift + P | Proxy |
| Ctrl + Shift + I | Intruder |
| Ctrl + Shift + R | Repeater |

---

## Settings

Two types:
- **Global/User settings** — apply to entire Burp installation
- **Project settings** — apply to current session only (lost on close in Community)

Notable settings locations:
- Cookie jar → Sessions category
- Keybindings → Hotkeys sub-category
- Update behavior → Suite category
- Client-side TLS certificates → can be overridden per project

---

## Proxy Setup with FoxyProxy

FoxyProxy is a Firefox extension that switches proxy settings with one click.

**Configure FoxyProxy for Burp:**
1. Install FoxyProxy Standard from Firefox Add-ons
2. Click FoxyProxy icon → Options
3. Add new proxy:
   - Title: Burp
   - Proxy IP: 127.0.0.1
   - Port: 8080
4. Save

**To intercept traffic:**
1. Start Burp Suite
2. Click FoxyProxy → select Burp profile
3. Enable Intercept in Proxy tab
4. Browse to target — browser hangs, request appears in Burp

**Turn intercept off** when not actively intercepting or your browser won't load anything.

---

## Installing Burp CA Certificate (for HTTPS)

Required to intercept HTTPS traffic without certificate errors.

1. Enable FoxyProxy Burp profile
2. Browse to `http://burp/cert` — downloads `cacert.der`
3. Firefox → Settings → Privacy & Security → View Certificates → Import
4. Select cacert.der → check "Trust this CA to identify websites" → OK

---

## Target Tab

Three sub-tabs:

**Site map** — tree view of all pages visited while proxy is active. Useful for API endpoint discovery.

**Issue definitions** — full list of vulnerabilities Burp Pro scanner checks for. Useful as a reference even in Community edition.

**Scope settings** — define which targets to include/exclude. Controls what gets proxied and logged.

**Setting scope:**
- Target tab → right-click target → Add to Scope
- Proxy settings → Intercept Client Requests → check "And URL Is in target scope"
- This filters out noise from unrelated traffic

---

## Proxy Key Behaviors

- Captures and logs all requests by default even when intercept is off
- HTTP history and WebSocket history tabs show all captured traffic
- Right-click any request to send to Repeater, Intruder, Decoder, Comparer, Sequencer, or Organizer
- Match and Replace — use regex to automatically modify requests/responses

---

## Burp Browser

Built-in Chromium pre-configured to use the proxy. Avoids manual FoxyProxy setup.

**If running as root on Linux (Kali):**
- Sandbox error will prevent it from launching
- Fix: Settings → Tools → Burp's browser → Allow Burp's browser to run without a sandbox
- Security risk — only use for lab environments

---

## Send to Shortcuts

| Shortcut | Action |
|---|---|
| Ctrl + R | Send to Repeater |
| Ctrl + I | Send to Intruder |
| Ctrl + O | Send to Organizer |

Right-click on any request in Proxy, Repeater, or anywhere else to send to other modules.
