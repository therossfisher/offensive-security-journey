# Protocols and Servers 2

**THM Room:** Protocols and Servers 2  
**Path:** Jr Penetration Tester / Network Security  
**Completed:** June 2026

---

## Core Concept
Cleartext protocols are inherently insecure. The three fundamental attacks against them are sniffing, MITM, and password attacks. The CIA triad maps directly to the attack triad DAD — Disclosure, Alteration, Destruction.

---

## The Three Attacks

### 1. Sniffing (Network Packet Capture)
Captures cleartext traffic to read credentials and data.

**Violates:** Confidentiality → Disclosure

**Still relevant against:**
- Internal corporate networks without encryption
- Legacy/IoT/embedded devices
- Misconfigured services where TLS exists but isn't enforced
- Wireless networks within range
- Post-MITM downgrade attacks

**Tools:**
| Tool | Type | Use |
|---|---|---|
| `tcpdump` | CLI | Lightweight, scriptable, default on Linux |
| `Wireshark` | GUI | Powerful filtering, protocol dissection, visualization |
| `tshark` | CLI | Wireshark engine in terminal, good for automation |
| `tcpflow` | CLI | Reassembles TCP streams |
| `ngrep` | CLI | Pattern matching in network traffic |
| `NetworkMiner` | GUI | Extracts files and images from captures |

**Key tcpdump commands:**
```bash
sudo tcpdump port 110 -A          # POP3 traffic in ASCII
sudo tcpdump port 23 -A           # Telnet traffic
sudo tcpdump port 21 -A           # FTP traffic
sudo tcpdump port 80 -A           # HTTP traffic
sudo tcpdump host 10.10.10.5 -A   # Traffic to/from specific host
sudo tcpdump -w capture.pcap      # Write to file
tcpdump -r capture.pcap -A        # Read capture file
```

**Wireshark display filters:**
```
pop        # POP3 traffic
imap       # IMAP traffic
ftp        # FTP traffic
http       # HTTP traffic
```

**Mitigations:** TLS encryption on all services, network segmentation, 802.1X port authentication, encrypted VLANs, zero trust architecture

---

### 2. Man-in-the-Middle (MITM)
Attacker positions between two parties, intercepting and potentially altering traffic.

**Violates:** Integrity → Alteration

**Attack techniques:**
| Technique | How It Works |
|---|---|
| ARP Spoofing | Forged ARP messages associate attacker MAC with gateway/target IP |
| DNS Spoofing | False DNS responses redirect victims to attacker-controlled servers |
| Rogue Access Points | Fake WiFi AP — all traffic flows through attacker |
| BGP Hijacking | False BGP routes redirect internet traffic (sophisticated, targeted) |
| SSL Stripping | Downgrades HTTPS to HTTP — victim may not notice |
| Fake Certificates | Attacker presents their own cert, intercepts both sides |

**MITM Tools:**
- **Bettercap** — modern successor to Ettercap, actively maintained, ARP/DNS spoofing, HTTP/HTTPS proxying
- **Ettercap** — classic LAN MITM tool, still functional
- **mitmproxy** — interactive HTTPS proxy for inspecting/modifying traffic
- **Responder** — Windows/AD environments, exploits LLMNR and NBT-NS for credential capture

**Modern defenses:**
- HSTS — browsers only connect via HTTPS, prevents SSL stripping
- Certificate Transparency — CAs must log all issued certs to public logs
- Certificate Pinning — apps specify exact valid certs, prevents CA compromise attacks
- DANE — uses DNSSEC to publish cert info in DNS records

---

### 3. Password Attacks

**Violates:** Confidentiality → Disclosure

**Attack types:**
| Type | Description |
|---|---|
| Password Guessing | Uses target-specific knowledge (pet names, birthdays, sports teams) |
| Dictionary Attack | Common words from wordlists |
| Brute Force | All possible combinations — exhaustive but slow |
| Credential Stuffing | Leaked username/password pairs tried across other services |
| Password Spraying | Few common passwords tried across many accounts — evades lockout |
| Hybrid Attack | Dictionary + patterns (Summer2024, P@ssw0rd) |

**Wordlists:**
- `/usr/share/wordlists/rockyou.txt` — classic breach list
- `/usr/share/seclists/` — collection for different purposes
- Custom wordlists tailored to target are most effective

---

## TLS — Transport Layer Security

### Version History
| Version | Status |
|---|---|
| SSL 2.0/3.0 | Deprecated — never use |
| TLS 1.0/1.1 | Deprecated 2021 — major browsers dropped support |
| TLS 1.2 | Still widely used, secure when properly configured |
| TLS 1.3 | Current standard — faster, forward secrecy by default |

### TLS 1.3 Improvements Over 1.2
- 1-RTT handshake (vs 2-RTT) — faster connections
- Forward secrecy by default — past sessions can't be decrypted if key is later compromised
- Simplified cipher suites — weak algorithms removed
- More of the handshake is encrypted

### Implicit TLS vs STARTTLS
| Type | How It Works | Vulnerability |
|---|---|---|
| Implicit TLS | Encryption starts immediately on dedicated port | More secure |
| STARTTLS | Connection starts cleartext, upgrades to TLS on same port | Vulnerable to downgrade attacks |

### Secure Protocol Reference

| Protocol | Cleartext Port | Secured | Secure Port | Type |
|---|---|---|---|---|
| HTTP | 80 | HTTPS | 443 | Implicit TLS |
| FTP | 21 | FTPS | 990 | Implicit TLS |
| SMTP | 25 | SMTPS | 465 | Implicit TLS |
| SMTP | 25 | SMTP Submission | 587 | STARTTLS |
| POP3 | 110 | POP3S | 995 | Implicit TLS |
| IMAP | 143 | IMAPS | 993 | Implicit TLS |
| Telnet | 23 | SSH | 22 | SSH encryption |
| FTP | 21 | SFTP | 22 | SSH encryption |

### DNS over TLS
- **DoT** (DNS over TLS) — port 853
- **DoH** (DNS over HTTPS) — port 443
- Both prevent eavesdropping on DNS lookups

---

## SSH

- Port 22 — replaced Telnet entirely for remote administration
- Encrypts all traffic including credentials

### Authentication Methods
| Method | Security | Notes |
|---|---|---|
| Password | Vulnerable to brute force | Simplest, not recommended for production |
| Public Key | Recommended | Private key stays local, public key on server |
| Certificate-based | Enterprise scale | CA signs keys, allows expiration/revocation |
| MFA | Strongest | Combines key + OTP |

### Key Generation
```bash
# Recommended — Ed25519
ssh-keygen -t ed25519 -C "email@example.com"

# Fallback — RSA 4096
ssh-keygen -t rsa -b 4096 -C "email@example.com"

# Copy public key to server
ssh-copy-id user@TARGET_IP
```

### Useful SSH Options
```bash
ssh -p 2222 user@TARGET_IP              # Non-standard port
ssh -i ~/.ssh/custom_key user@TARGET_IP  # Specific key
ssh -J bastion user@internal            # Jump through bastion host
ssh -L 8080:localhost:80 user@TARGET_IP # Local port forwarding
ssh -D 9050 user@TARGET_IP             # SOCKS proxy
ssh user@TARGET_IP "cat /etc/passwd"   # Single command
```

### SSH Config Shortcut (~/.ssh/config)
```
Host webserver
    HostName TARGET_IP
    User mark
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```
Then just: `ssh webserver`

### File Transfer
```bash
# SFTP — recommended
sftp user@TARGET_IP

# SCP — being deprecated but still works
scp user@TARGET_IP:/remote/file ./local/    # Download
scp local/file user@TARGET_IP:/remote/     # Upload

# rsync — best for large transfers
rsync -avz -e ssh /local/ user@TARGET_IP:/remote/
```

### SSH Hardening (sshd_config)
```bash
PasswordAuthentication no      # Disable password auth after setting up keys
PermitRootLogin no             # Force non-root login
AllowUsers mark admin          # Restrict which accounts can SSH
```

---

## Password Attack Mitigations

| Defense | How It Helps |
|---|---|
| Strong password policies | Length over complexity (NIST SP 800-63B) |
| Account lockout | Stops brute force — bypassed by spraying |
| Rate limiting / throttling | Slows automated tools |
| MFA | Most effective — second factor defeats stolen credentials |
| Breached password detection | Block known compromised passwords at login |
| Passkeys (FIDO2/WebAuthn) | Eliminates passwords entirely |
| Behavioral analysis | Detect anomalous login patterns |

---

## Defensive Checklist

- [ ] All services use TLS 1.2+ with strong cipher suites
- [ ] Cleartext protocols (Telnet, FTP, HTTP) disabled or isolated
- [ ] SSH uses key-based auth with password auth disabled
- [ ] Strong password policies with breached password detection
- [ ] Account lockout or rate limiting on all auth endpoints
- [ ] MFA enabled for sensitive systems
- [ ] Network segmentation limits sniffing blast radius
- [ ] HSTS enabled on web applications
- [ ] Logging and monitoring for auth anomalies
