# Gobuster Cheat Sheet

Directory, file, DNS, and virtual host enumeration tool. Pre-installed on Kali.

---

## Modes

| Mode | What It Does |
|---|---|
| `dir` | Brute force directories and files on a web server |
| `dns` | Brute force DNS subdomains |
| `vhost` | Brute force virtual hosts via Host header |

---

## Key Flags

| Flag | Description |
|---|---|
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-x` | File extensions to append (multiplies requests) |
| `-t` | Threads — default 10, use 50 for speed |
| `-o` | Save output to file |
| `-b` | Blacklist status codes — use `""` to clear defaults |
| `-s` | Whitelist status codes — only show these |
| `--timeout` | Request timeout e.g. `--timeout 10s` |
| `-k` | Skip TLS certificate verification |
| `-q` | Quiet — only show results, no banner |
| `-r` | Follow redirects |
| `-d` | Domain for DNS mode |
| `--append-domain` | Append domain to wordlist entries in vhost mode |
| `--exclude-length` | Exclude responses of specific length (removes false positives) |

---

## Wordlists

| Wordlist | Size | Best For |
|---|---|---|
| `/usr/share/wordlists/dirb/common.txt` | ~4K | Quick first pass |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` | ~87K | Fast, thorough |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | ~220K | Most thorough |
| `/usr/share/seclists/Discovery/Web-Content/common.txt` | ~4.7K | Best general purpose |
| `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` | ~30K | Good all-around |
| `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | 5K | Subdomain enum |

**Strategy:** Start with `dirb/common.txt` for speed. Move to `dirbuster/medium` if nothing useful comes back.

---

## dir Mode — Directory and File Enumeration

```bash
# Fast first pass
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -t 50

# With file extensions
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -x php,html,txt,bak -t 50

# Save output
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -t 50 -o gobuster.txt

# HTTPS with self-signed cert
gobuster dir -u https://TARGET -w /usr/share/wordlists/dirb/common.txt -k -t 50

# Only show specific status codes
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -s 200,301,302,307 -b "" -t 50

# Find middleware-protected routes (307 redirects — Next.js CVE-2025-29927)
gobuster dir -u http://TARGET:3000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -s 307,302 -b "" -t 50

# Non-standard port
gobuster dir -u http://TARGET:8080 -w /usr/share/wordlists/dirb/common.txt -t 50

# With cookie (session-based auth)
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -c "session=TOKEN" -t 50
```

---

## dns Mode — Subdomain Enumeration

```bash
# Basic subdomain scan
gobuster dns -d TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Show IPs subdomains resolve to
gobuster dns -d TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -i

# Force enumeration even with wildcard DNS
gobuster dns -d TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard

# Use custom DNS resolver
gobuster dns -d TARGET.com -w WORDLIST -r 8.8.8.8
```

---

## vhost Mode — Virtual Host Enumeration

```bash
# Basic vhost scan
gobuster vhost -u http://TARGET -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain

# Filter false positives by response length
gobuster vhost -u http://TARGET --domain TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

**vhost vs dns:** dns uses real DNS lookups. vhost manipulates the Host header — finds virtual hosts not in public DNS.

---

## Speed Reference

| Threads | Use When |
|---|---|
| 10 | Default, stable target |
| 50 | Normal HTB/THM use |
| 100 | Fast target, noise doesn't matter |

Adding `-x` extensions multiplies requests — `-x php,html,txt,bak` = 4x the requests. Only add when you have a reason.

---

## Status Code Reference

| Code | Meaning | Action |
|---|---|---|
| 200 | Found, accessible | Investigate |
| 301/302 | Redirect | Follow it |
| 307 | Temporary redirect | Check for middleware protection |
| 401 | Unauthorized | Try credentials |
| 403 | Forbidden | Exists but can't access — note it |
| 404 | Not found | Ignore |
| 500 | Server error | Investigate — may reveal info |

---

## High Value Findings

When these appear, investigate immediately:

```
/admin          /administrator      /manage
/login          /signin             /auth
/api            /api/v1             /api/v2
/upload         /uploads            /files
/backup         /backups            /bak
/config         /configuration      /settings
/includes       /inc                /lib
/data           /database           /db
/console        /dashboard          /panel
/phpinfo.php    /info.php           /test.php
/.env           /.git               /web.config
/server-status  /server-info        /nginx_status
```

---

## Common Extension Sets

```bash
# PHP site
-x php,phtml,php3

# ASP.NET / IIS
-x asp,aspx,config,bak

# Backup hunting
-x bak,old,backup,txt,zip

# All useful at once (slower)
-x php,html,txt,bak,zip,asp,aspx,config
```

---

## Fixing Common Errors

```bash
# Error: status-codes and status-codes-blacklist both set
# Fix: clear the blacklist when using -s
gobuster dir -u http://TARGET -w WORDLIST -s 307,302 -b ""

# Timeouts on slow targets
gobuster dir -u http://TARGET -w WORDLIST --timeout 10s -t 20

# TLS certificate errors
gobuster dir -u https://TARGET -w WORDLIST -k
```

---

## Full Workflow

```bash
# Step 1 — Fast first pass
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -t 50

# Step 2 — Nothing useful, go bigger
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50

# Step 3 — Hunt backup and config files
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -x bak,txt,zip,config -t 50

# Step 4 — Enumerate subdomains
gobuster dns -d TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard

# Step 5 — Check for virtual hosts
gobuster vhost -u http://TARGET --domain TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```
