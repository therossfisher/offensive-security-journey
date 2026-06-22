# John the Ripper

Password hash cracking tool. Tests passwords from a wordlist by hashing them and comparing to the target hash. Pre-installed on Kali as Jumbo John — the extended community version that includes extra tools like `zip2john`, `rar2john`, `ssh2john`, and `unshadow`.

**Core concept:** Hashing is one-way — you can't reverse a hash mathematically. John works around this by hashing thousands of candidate passwords and comparing results. If a hash matches, the password is found. This is a dictionary attack, not decryption.

---

## How to Identify a Hash

Before cracking, you need to know what you're dealing with.

```bash
# Hash length is a fast first clue
# MD5      = 32 chars
# SHA1     = 40 chars
# SHA256   = 64 chars
# SHA512   = 128 chars

# hash-id.py — interactive identifier
hash-id.py

# Or pipe it directly
echo "5f4dcc3b5aa765d61d8327deb882cf99" | hash-id.py

# List all formats John supports
john --list=formats

# Search for a specific format
john --list=formats | grep -iF "md5"
```

**Note on format prefixes:** Standard standalone hashes (MD5, SHA1, SHA256 etc.) need the `raw-` prefix in John — e.g. `raw-md5`, `raw-sha256`. Hashes from system files (like `/etc/shadow`) use format names without the prefix — e.g. `sha512crypt`, `md5crypt`.

---

## Basic Syntax

```bash
john [options] [file path]
```

```bash
# Auto-detect hash type and crack (unreliable — try if you're unsure)
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Specify format explicitly (preferred)
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Show cracked passwords (reads from pot file)
john --show hash.txt
john --show --format=raw-md5 hash.txt

# Force re-crack ignoring pot file cache
john --pot=/dev/null --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## Common Hash Formats

| Hash Type | Where You'll Find It | John Format |
|---|---|---|
| MD5 | Web apps, old Linux shadow | `raw-md5` |
| SHA1 | Web apps | `raw-sha1` |
| SHA256 | Web apps, CTFs | `raw-sha256` |
| Whirlpool | Less common, CTFs | `whirlpool` |
| NTHash / NTLM | Windows SAM, NTDS.dit | `nt` |
| NetNTLMv2 | Responder captures | `netntlmv2` |
| SHA512crypt | Modern Linux /etc/shadow | `sha512crypt` |
| MD5crypt | Older Linux /etc/shadow | `md5crypt` |
| bcrypt | Modern web apps | `bcrypt` |

---

## Wordlists

```bash
# Best general purpose — start here
/usr/share/wordlists/rockyou.txt

# May need to decompress first
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# See what else is available
ls /usr/share/wordlists/

# SecLists — worth installing for more coverage
sudo apt install seclists
ls /usr/share/seclists/Passwords/
```

---

## Cracking Modes

### Wordlist Mode
Standard dictionary attack. Hashes every word in the list and compares.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

### Wordlist Mode with Rules
Applies mutation rules to every word in the list. Catches things like `password` → `Password1!` that wouldn't be in the wordlist directly.

```bash
# Use default built-in rules
john --wordlist=/usr/share/wordlists/rockyou.txt --rules hash.txt

# Use a specific ruleset
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=jumbo hash.txt

# Use a custom rule by name (see Custom Rules section)
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=MyRuleName hash.txt
```

### Single Crack Mode
Uses the username itself as the base to generate password candidates via word mangling. No wordlist needed — John derives candidates from the username (e.g. `Markus` → `Markus1`, `MArkus`, `Markus!` etc.).

**File format requirement:** Hash must be prepended with `username:` for this to work.

```bash
# File must contain: username:hash
# Example: joker:7bf6d9bb82bed1302f331fc6b816aada

john --single --format=raw-md5 hash.txt
```

**GECOS compatibility:** When cracking `/etc/shadow` hashes, John can also pull the full name and other info from the GECOS field in `/etc/passwd` (5th colon-separated field) to enrich its candidate list. This is why `unshadow` combines both files before cracking.

### Incremental / Brute Force Mode
Tries every possible character combination. Slow — use as last resort.

```bash
john --incremental hash.txt
```

---

## Custom Rules

Defined in `/etc/john/john.conf` (Kali package install) or `/opt/john/john.conf` (built from source).

### Rule Syntax

```
[List.Rules:RuleName]
MODIFIERS"[CHARACTER_SET]"
```

**Modifiers:**
- `Az` — append to end of word
- `A0` — prepend to beginning of word
- `c` — capitalize character at that position

**Character sets:**
- `[0-9]` — digits
- `[A-Z]` — uppercase letters
- `[a-z]` — lowercase letters
- `[A-z]` — both upper and lowercase
- `[!£$%@]` — specific symbols

### Common Rule Examples

| Goal | Rule |
|---|---|
| Append all capital letters to end | `Az"[A-Z]"` |
| Append all digits to end | `Az"[0-9]"` |
| Prepend digits to start | `A0"[0-9]"` |
| Capitalize first letter + append digit + symbol | `cAz"[0-9][!£$%@]"` |

### Example: Password Complexity Pattern
Target: `Polopassword1!` (capital first, word, digit, symbol)

```
[List.Rules:PoloPassword]
cAz"[0-9][!£$%@]"
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=PoloPassword hash.txt
```

**Call any custom rule with:** `--rules=RuleName`
**Call THMRules:** `--rules=THMRules`

---

## Cracking System Hashes

### Linux /etc/shadow

Shadow file hashes can't be fed directly into John — they need to be combined with `/etc/passwd` first using `unshadow`. This gives John the username context it needs.

```bash
# Combine passwd and shadow
unshadow /etc/passwd /etc/shadow > unshadowed.txt

# Or use extracted lines from each file
unshadow local_passwd local_shadow > unshadowed.txt

# Crack
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```

### Windows NTHash / NTLM

Obtained by dumping the SAM database (Mimikatz, secretsdump, etc.) or from NTDS.dit on a domain controller.

```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm_hash.txt
```

**Pass-the-Hash note:** You don't always need to crack NTLM hashes — you can often use them directly in pass-the-hash attacks with tools like `evil-winrm`, `crackmapexec`, or `impacket`. Cracking is one option, not the only one.

### NetNTLMv2 (Responder Captures)

Captured during LLMNR/NBT-NS poisoning with Responder. Different format from NTHash — can't be used for pass-the-hash, must be cracked.

```bash
john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt responder_hash.txt
```

---

## Cracking File Archives

### ZIP Files

```bash
# Step 1 — convert to john-readable hash
zip2john secure.zip > zip_hash.txt

# Step 2 — crack
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# Step 3 — show password
john --show zip_hash.txt
```

### RAR Files

```bash
# Step 1 — convert to john-readable hash
rar2john secure.rar > rar_hash.txt

# Step 2 — crack
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt

# Step 3 — extract once you have the password
unrar x secure.rar
```

**Install unrar if needed:**
```bash
sudo apt install unrar
```

---

## Cracking SSH Private Key Passphrases

If you find an `id_rsa` private key on a target, it may be passphrase-protected. John can crack the passphrase.

```bash
# Step 1 — convert to john-readable hash
ssh2john id_rsa > id_rsa_hash.txt

# On some systems ssh2john is a Python script
python3 /usr/share/john/ssh2john.py id_rsa > id_rsa_hash.txt

# Step 2 — crack
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt

# Step 3 — show password
john --show id_rsa_hash.txt

# Step 4 — use the key
chmod 600 id_rsa
ssh -i id_rsa user@TARGET_IP
# Enter cracked passphrase when prompted
```

---

## Session Management

John saves cracking progress automatically. If a session is interrupted, you can resume.

```bash
# Resume most recent session
john --restore

# Resume named session
john --restore=mysession

# Start a named session
john --session=mysession --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

---

## Pot File

John stores cracked hashes in `~/.john/john.pot`. This is why re-running John on an already-cracked hash shows nothing — it skips hashes it's already solved.

```bash
# View cracked results
john --show hash.txt

# Force re-crack ignoring the pot file
john --pot=/dev/null --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# View pot file directly
cat ~/.john/john.pot
```

---

## John vs Hashcat

| | John | Hashcat |
|---|---|---|
| Speed | Slower | Much faster (GPU) |
| Ease of use | Simpler syntax | More flags to learn |
| Built-in converters | Yes (zip2john, ssh2john etc.) | No |
| Best for | Lab environments, quick cracks, file archives | Large wordlists, real engagements with GPU |

```bash
# Hashcat equivalents for common types
hashcat -m 0    hash.txt rockyou.txt   # MD5
hashcat -m 100  hash.txt rockyou.txt   # SHA1
hashcat -m 1400 hash.txt rockyou.txt   # SHA256
hashcat -m 1000 hash.txt rockyou.txt   # NTHash
hashcat -m 5600 hash.txt rockyou.txt   # NetNTLMv2
hashcat -m 3200 hash.txt rockyou.txt   # bcrypt
hashcat -m 1800 hash.txt rockyou.txt   # SHA512crypt
```

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `No password hashes loaded` | Wrong format, bad file encoding, or John can't parse the hash | Check hash length, run `hash-id.py`, verify file with `cat -A` for `^M` characters |
| `python\r: No such file or directory` | Windows line endings in script shebang | `sed -i 's/\r//' scriptname.py` |
| John runs but finds nothing | Hash already in pot file | Run with `--pot=/dev/null` to force re-crack |
| Single crack mode finds nothing | Username:hash format missing | File must be `username:hash`, not just the hash |

---

## Pentest Workflow Reference

```
1. Get hash (from shadow file, SAM dump, Responder, CTF file, etc.)
2. Identify hash type — hash-id.py or length
3. Choose attack:
   - Known wordlist hit likely?  → wordlist mode
   - Password policy known?      → wordlist + custom rules
   - Username-based password?    → single crack mode
   - File archive?               → zip2john / rar2john first
   - SSH key?                    → ssh2john first
   - Have GPU available?         → hashcat instead
4. john --format=X --wordlist=rockyou.txt hash.txt
5. john --show hash.txt
```

---

## Resources

- John the Ripper wiki: https://openwall.info/wiki/john
- Custom rules reference: https://openwall.info/wiki/john/rules
- GTFOBins (for post-crack priv esc): https://gtfobins.github.io
- SecLists (more wordlists): https://github.com/danielmiessler/SecLists
