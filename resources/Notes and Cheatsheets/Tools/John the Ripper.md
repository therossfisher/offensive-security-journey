# John the Ripper Cheat Sheet

Password hash cracking tool. Tests passwords from a wordlist by hashing them and comparing to the target hash. Pre-installed on Kali.

---

## Basic Usage

```bash
# Crack a hash file (auto-detects hash type)
john hash.txt

# Crack with specific wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Show cracked passwords
john --show hash.txt

# Show cracked passwords for specific hash file
john --show --format=netntlmv2 hash.txt
```

---

## Specifying Hash Format

John usually auto-detects but sometimes needs help:

```bash
# List all supported formats
john --list=formats

# Specify format manually
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## Common Hash Formats

| Hash Type | Where Found | John Format |
|---|---|---|
| NetNTLMv2 | Responder captures | `netntlmv2` |
| NTHash | Windows SAM/NTDS | `nt` |
| MD5 | Web apps, Linux shadow | `raw-md5` |
| SHA1 | Web apps | `raw-sha1` |
| bcrypt | Modern web apps | `bcrypt` |
| SHA512crypt | Linux /etc/shadow | `sha512crypt` |
| MD5crypt | Linux /etc/shadow | `md5crypt` |

---

## Wordlists

```bash
# Best general purpose
/usr/share/wordlists/rockyou.txt

# Check what's available
ls /usr/share/wordlists/
```

---

## Rules — Mutate Wordlist Entries

Rules modify wordlist entries to try variations:

```bash
# Use built-in rules
john --wordlist=rockyou.txt --rules hash.txt

# Specific ruleset
john --wordlist=rockyou.txt --rules=jumbo hash.txt
```

Rules try things like: `password` → `Password`, `password1`, `P@ssword`, `PASSWORD` etc.

---

## Saving and Resuming

```bash
# John saves progress automatically
# Resume an interrupted session
john --restore

# Restore specific session
john --restore=SESSION_NAME
```

---

## Creating Hash Files

```bash
# Single hash
echo "HASH_STRING" > hash.txt

# NetNTLMv2 format (from Responder)
echo "USERNAME::DOMAIN:CHALLENGE:RESPONSE:BLOB" > hash.txt

# Linux shadow file entries work directly
sudo cat /etc/shadow > shadow.txt
john shadow.txt
```

---

## Useful One-Liners

```bash
# Crack NetNTLMv2 from Responder
john --wordlist=/usr/share/wordlists/rockyou.txt --format=netntlmv2 hash.txt

# Crack Linux shadow file
john --wordlist=/usr/share/wordlists/rockyou.txt /etc/shadow

# Check if already cracked
john --show hash.txt

# Crack with rules (tries password mutations)
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt
```

---

## Hashcat Alternative (Faster — Uses GPU)

```bash
# NetNTLMv2
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

# NTHash
hashcat -m 1000 hash.txt /usr/share/wordlists/rockyou.txt

# MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

Hashcat is significantly faster than John when a GPU is available. John is fine for lab environments.

---

## Pentest Notes

- John auto-detects hash type most of the time — try without `--format` first
- If cracking fails: try rules, try a bigger wordlist, try hashcat with GPU
- NetNTLMv2 from Responder ≠ NTHash from SAM — different formats, different attacks
- Cracked passwords go into John's pot file — `~/.john/john.pot`
- `--show` reads from the pot file — run it after cracking to see results
