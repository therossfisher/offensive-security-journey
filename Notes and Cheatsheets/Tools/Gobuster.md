# Gobuster Cheat Sheet

## Syntax
```
gobuster [mode] [options]
```

---

## Modes

| Mode | Description |
|---|---|
| `dir` | Directory/file brute force |
| `dns` | DNS subdomain brute force |
| `vhost` | Virtual host brute force |

---

## Key Switches

| Switch | Description |
|---|---|
| `-u` | Target URL |
| `-w` | Wordlist path |
| `-x` | File extensions to append (e.g. `-x php,html,txt`) |
| `-t` | Number of threads (default 10, increase for speed) |
| `-o` | Save output to file |
| `-b` | Blacklist status codes (e.g. `-b 404,403`) |
| `-s` | Whitelist status codes (e.g. `-s 200,301`) |
| `--timeout` | Request timeout (e.g. `--timeout 10s`) |
| `-k` | Skip TLS certificate verification |
| `-q` | Quiet mode — only show results |

---

## Common Wordlists (Kali)

| Wordlist | Use Case |
|---|---|
| `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` | Fast, common directories |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | More thorough |
| `/usr/share/wordlists/dirb/common.txt` | Quick and dirty recon |
| `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` | Good all-around (SecLists) |

---

## Common Combos

```bash
# Basic directory scan
gobuster dir -u http://10.10.10.5 -w /usr/share/wordlists/dirb/common.txt

# Scan for PHP files
gobuster dir -u http://10.10.10.5 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php

# Multiple extensions
gobuster dir -u http://10.10.10.5 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt

# Faster with more threads
gobuster dir -u http://10.10.10.5 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50

# DNS subdomain brute force
gobuster dns -d target.com -w /usr/share/wordlists/dirb/common.txt
```

---

## High Value Findings
When these show up in results, investigate immediately:
- `/admin` `/administrator`
- `/uploads` `/upload`
- `/backup` `/backups`
- `/config` `/configuration`
- `/data` `/database`
- `/includes` `/inc`
- `/api`
- `/login` `/signin`

---

## Notes
- Start with `small` wordlist, move to `medium` if nothing interesting shows up
- Add `-x php` when you know the site runs PHP
- If getting timeouts use `--timeout 10s -t 50`
