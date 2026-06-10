# Protocols and Servers

**THM Room:** Protocols and Servers  
**Path:** Jr Penetration Tester / Network Security  
**Completed:** June 2026

---

## Core Concept
Every protocol covered in this room transmits data in **cleartext by default** — including authentication credentials. Anyone with network access can capture usernames, passwords, email content, and file transfers.

---

## Protocol Quick Reference

| Protocol | Port | Purpose | Security | Secure Alternative | Secure Port |
|---|---|---|---|---|---|
| Telnet | 23 | Remote access | Cleartext | SSH | 22 |
| HTTP | 80 | Web | Cleartext | HTTPS | 443 |
| FTP | 21 | File transfer | Cleartext | SFTP or FTPS | 22 (SFTP), 990 (FTPS) |
| SMTP | 25 | Email sending (MTA) | Cleartext | SMTPS / STARTTLS | 465 (SMTPS), 587 (Submission) |
| POP3 | 110 | Email retrieval | Cleartext | POP3S | 995 |
| IMAP | 143 | Email sync | Cleartext | IMAPS | 993 |

---

## Telnet

- Remote terminal access protocol — port 23
- All traffic including credentials sent in cleartext
- Replaced by SSH on modern systems
- **Still found on:** legacy systems, old network equipment, IoT devices, misconfigured servers
- **Pentest value:** Finding open port 23 = significant finding (legacy system or misconfiguration)
- **Useful as a testing tool:** `telnet TARGET PORT` connects to any TCP port for manual protocol interaction

```bash
telnet 10.10.10.5 80    # interact with HTTP manually
telnet 10.10.10.5 25    # interact with SMTP manually
telnet 10.10.10.5 21    # interact with FTP manually
```

---

## HTTP

- Port 80 (cleartext), Port 443 (HTTPS/TLS)
- Commands are identical between HTTP and HTTPS — HTTPS just wraps them in TLS
- **Version notes:**
  - HTTP/1.1 — text-based, manually testable with Telnet/Netcat
  - HTTP/2 — binary, multiplexed, not manually testable
  - HTTP/3 — uses QUIC (UDP port 443)

**Manual HTTP request:**
```
GET /index.html HTTP/1.1
host: telnet
[blank line]
```

**What response headers reveal:**
- `Server: nginx/1.18.0 (Ubuntu)` — web server software, version, and OS
- Security-conscious admins suppress this — finding it is worth noting in a pentest

**Common web servers:** Nginx, Apache, IIS, LiteSpeed, Caddy

---

## FTP

- Port 21 (control channel), Port 20 (data channel — active mode)
- Credentials and data sent in cleartext
- Dual-connection architecture: separate TCP connections for commands and data transfer
- **Two modes:**
  - **Active** — server initiates data connection back to client (fails behind NAT/firewalls)
  - **Passive** — client initiates both connections (firewall-friendly, default for most clients)

**Key FTP commands:**
```bash
USER username     # authenticate
PASS password     # password
SYST              # system type
PASV              # switch to passive mode
LIST              # list files
TYPE A            # ASCII mode
TYPE I            # binary mode
STAT              # server status
```

**Anonymous FTP — always try this:**
```bash
USER anonymous
PASS anything@example.com
```
Anonymous access may expose sensitive files, config backups, or allow file uploads.

**Secure alternatives:** SFTP (port 22), FTPS (port 990/21+STARTTLS), SCP

---

## Email Architecture

| Component | Role |
|---|---|
| **MUA** (Mail User Agent) | Email client — Outlook, Thunderbird, webmail |
| **MSA** (Mail Submission Agent) | Receives mail from MUA, checks for errors |
| **MTA** (Mail Transfer Agent) | Routes and delivers mail between servers |
| **MDA** (Mail Delivery Agent) | Stores email in recipient's mailbox |

**Flow:** MUA → MSA → MTA → MTA (recipient) → MDA → MUA (recipient)

---

## SMTP

- Port 25 — server-to-server (MTA to MTA), often blocked by ISPs
- Port 587 — client submission (MUA to MSA), requires auth, STARTTLS
- Port 465 — implicit TLS (SMTPS)

**Manual SMTP session:**
```
helo telnet
mail from: <sender@domain.com>
rcpt to: <recipient@domain.com>
data
subject: Test
Message body here
.
quit
```

**Why email spoofing works:** SMTP has no built-in sender verification. The `mail from:` address is accepted without proof of ownership. This is why phishing emails can appear to come from legitimate addresses.

**Modern mitigations:** SPF, DKIM, DMARC (covered in Protocols and Servers 2)

**Pentest value:**
- Test for open relay configurations
- Attempt email spoofing to assess security awareness
- Analyze email headers to trace phishing origins

---

## POP3

- Port 110 (cleartext), Port 995 (POP3S)
- Downloads email from MDA then optionally deletes from server
- "Download and delete" model — email stored locally, not synced across devices

**Key POP3 commands:**
```
USER username
PASS password
STAT              # returns: +OK [message count] [total size in octets]
LIST              # list messages with sizes
RETR n            # retrieve message n
DELE n            # mark message n for deletion
RSET              # unmark deletions
QUIT              # end session, execute deletions
```

**STAT response format:** `+OK nn mm` where nn = message count, mm = inbox size in bytes

**When to use POP3 over IMAP:** Offline access, single device, minimal server storage, local archiving

---

## IMAP

- Port 143 (cleartext), Port 993 (IMAPS)
- Emails remain on server — synced across all devices
- Read/unread status, folders, and flags synchronized everywhere
- Server-side search without downloading all messages

**Key IMAP commands (each requires a unique tag prefix):**
```
c1 LOGIN username password
c2 LIST "" "*"              # list all folders
c3 SELECT INBOX             # open folder read/write
c4 EXAMINE INBOX            # open folder read-only
c5 FETCH n BODY[]           # retrieve message n
c6 SEARCH criteria          # search messages
c7 STORE n +FLAGS (\Seen)   # mark as read
c8 LOGOUT
```

**Why IMAP is more valuable to attackers than POP3:**
- Persistent access — emails stay on server, attacker reads new mail indefinitely
- Full mailbox history — years of sensitive communications
- Password reset emails — pivot to other account takeovers
- Business email compromise — invoice fraud, impersonation
- Lateral movement — credentials and internal docs in email threads

---

## Pentest Findings Summary

| Finding | Severity | Why It Matters |
|---|---|---|
| Open Telnet (port 23) | High | Cleartext credentials, legacy system indicator |
| FTP with anonymous login | High | Potential file exposure or upload vector |
| Plain HTTP with sensitive data | Medium-High | Credentials and content visible on network |
| SMTP open relay | High | Can be used to send spam/phishing as the organization |
| POP3/IMAP on cleartext ports | Medium | Credential capture via sniffing |
| Server version disclosure in HTTP headers | Low-Medium | Enables targeted vulnerability research |

---

## What's Coming Next (Protocols and Servers 2)
- TLS encryption for all these protocols
- Sniffing attacks
- Man-in-the-middle attacks
- Password attacks
- SPF, DKIM, DMARC for email authentication
