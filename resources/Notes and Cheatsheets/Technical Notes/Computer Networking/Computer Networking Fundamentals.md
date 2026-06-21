# Networking Fundamentals — Study Notes

---

## OSI Model (7 Layers)

Mnemonic bottom→top: **P**lease **D**o **N**ot **T**hrow **S**pinach **P**izza **A**way

| Layer | Name | Function | Examples |
|-------|------|----------|---------|
| 7 | Application | Services/interfaces for apps | HTTP, FTP, DNS, SMTP, IMAP, POP3 |
| 6 | Presentation | Encoding, encryption, compression | Unicode, MIME, JPEG, PNG |
| 5 | Session | Establish/maintain/sync sessions | NFS, RPC |
| 4 | Transport | End-to-end communication, segmentation | TCP, UDP |
| 3 | Network | Logical addressing and routing | IP, ICMP, IPSec |
| 2 | Data Link | Data transfer between adjacent nodes | Ethernet (802.3), WiFi (802.11) |
| 1 | Physical | Physical transmission medium | Electrical, optical, wireless |

Know the numbers — terms like "layer 3 switch" and "layer 7 firewall" come up constantly.

---

## TCP/IP Model

Collapses OSI's 7 layers into 4 (sometimes shown as 5 with Physical included):

| TCP/IP Layer | Maps to OSI |
|-------------|-------------|
| Application | Layers 5, 6, 7 |
| Transport | Layer 4 |
| Internet | Layer 3 |
| Link | Layer 2 |

---

## IP Addressing

IPv4 = 32 bits, four octets (0–255 each)  
`192.168.1.0` = network address / `192.168.1.255` = broadcast address

**Subnet notation:** `255.255.255.0` = `/24` (leftmost 24 bits fixed across subnet)

### Private IP Ranges (RFC 1918)
| Range | CIDR |
|-------|------|
| 10.0.0.0 – 10.255.255.255 | 10/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168/16 |

Private IPs can't reach the internet directly — need a router with NAT and a public IP.

**Check IP on Linux:** `ifconfig` or `ip a s`  
**Check IP on Windows:** `ipconfig`

---

## UDP vs TCP

| Feature | UDP | TCP |
|---------|-----|-----|
| Connection | Connectionless | Connection-oriented |
| Reliability | No delivery confirmation | Acknowledged, sequenced |
| Speed | Faster | Slower (overhead) |
| Use case | Streaming, DNS, DHCP | Web, email, file transfer |
| Layer | 4 | 4 |

**Port numbers:** 1–65535 (2 octets; port 0 reserved) ≈ 65K ports

### TCP Three-Way Handshake
1. Client → Server: **SYN**
2. Server → Client: **SYN-ACK**
3. Client → Server: **ACK**

---

## Encapsulation

Each layer wraps the layer above's data in a header (and sometimes trailer):

```
Application Data
  → TCP/UDP adds header → Segment / Datagram
    → IP adds header → Packet
      → Ethernet/WiFi adds header+trailer → Frame
```

Reversed on the receiving end. Key terms:

| Layer | Data Unit |
|-------|-----------|
| Transport (TCP) | Segment |
| Transport (UDP) | Datagram |
| Network (IP) | Packet |
| Data Link | Frame |

---

## Key Supporting Protocols

### DHCP (Dynamic Host Configuration Protocol)
Automatically assigns IP, subnet mask, gateway, and DNS to devices.  
Uses **UDP** — server listens port **67**, client sends from port **68**

**DORA process:**
1. **Discover** — client broadcasts from `0.0.0.0` to `255.255.255.255`
2. **Offer** — DHCP server offers an IP
3. **Request** — client accepts the offer
4. **Acknowledge** — server confirms assignment

### ARP (Address Resolution Protocol)
Resolves IP addresses → MAC addresses on the same network segment.  
- ARP Request sent to broadcast MAC: `ff:ff:ff:ff:ff:ff`
- Target replies with its MAC address
- Sits between layer 2 and 3 (bridges MAC and IP addressing)

### ICMP (Internet Control Message Protocol)
Used for network diagnostics and error reporting. Layer 3.

| Tool | ICMP Type | Purpose |
|------|-----------|---------|
| `ping` | Echo Request (Type 8) / Echo Reply (Type 0) | Test connectivity, measure RTT |
| `traceroute` / `tracert` | Time Exceeded (Type 11) | Map route to destination |

**TTL (Time to Live):** Each router decrements TTL by 1. When TTL hits 0, router drops packet and sends ICMP Time Exceeded back. Traceroute exploits this to reveal each hop.

### NAT (Network Address Translation)
Allows many private IPs to share one public IP.  
Router maintains a translation table mapping internal IP:port → external IP:port.  
~65,000 simultaneous TCP connections possible (one per port number).

### Routing Protocols

| Protocol | Notes |
|----------|-------|
| OSPF | Open Shortest Path First — shares full network topology map |
| EIGRP | Cisco proprietary — hybrid approach |
| BGP | Border Gateway Protocol — the backbone of the Internet |
| RIP | Simple, small networks — routes by hop count |

---

## Core Application Protocols & Ports

| Protocol | Transport | Port | Purpose |
|----------|-----------|------|---------|
| DNS | UDP/TCP | 53 | Domain name → IP resolution |
| HTTP | TCP | 80 | Web pages |
| HTTPS | TCP | 443 | HTTP over TLS |
| FTP | TCP | 21 | File transfer |
| SFTP | TCP | 22 | Secure file transfer (over SSH) |
| FTPS | TCP | 990 | FTP over TLS |
| SMTP | TCP | 25 | Send email |
| SMTPS | TCP | 465/587 | SMTP over TLS |
| POP3 | TCP | 110 | Retrieve email (downloads, deletes from server) |
| POP3S | TCP | 995 | POP3 over TLS |
| IMAP | TCP | 143 | Retrieve email (syncs, keeps on server) |
| IMAPS | TCP | 993 | IMAP over TLS |
| TELNET | TCP | 23 | Remote terminal (plaintext — insecure) |
| SSH | TCP | 22 | Remote terminal (encrypted) |

---

## DNS Record Types

| Record | Purpose |
|--------|---------|
| A | Domain → IPv4 address |
| AAAA | Domain → IPv6 address |
| CNAME | Domain → another domain (alias) |
| MX | Mail server for a domain |

DNS uses **UDP port 53** by default; falls back to TCP port 53.

**Lookup from CLI:** `nslookup example.com`  
**WHOIS lookup:** `whois example.com` — shows registration info, registrant, dates

---

## HTTP Methods

| Method | Purpose |
|--------|---------|
| GET | Retrieve data from server |
| POST | Submit new data to server |
| PUT | Create or overwrite a resource |
| DELETE | Remove a resource |

---

## Protocol Commands Reference

### FTP
```
USER <username>
PASS <password>
ls          → LIST
get <file>  → RETR
put <file>  → STOR
type ascii  → switch to ASCII mode
quit
```

### SMTP (port 25)
```
HELO / EHLO   → initiate session
MAIL FROM:    → sender address
RCPT TO:      → recipient address
DATA          → begin message body
.             → end message (period on its own line)
QUIT
```

### POP3 (port 110)
```
USER <username>
PASS <password>
STAT          → message count + size
LIST          → list messages
RETR <n>      → retrieve message n
DELE <n>      → mark message n for deletion
QUIT
```

### IMAP (port 143)
```
A LOGIN <user> <pass>
B SELECT inbox
C FETCH <n> body[]    → retrieve message n
D LOGOUT
```
*(IMAP requires a tag prefix before each command — A, B, C, D, etc.)*

---

## Connecting to Services with Telnet

```bash
telnet <IP> <port>

# Examples:
telnet 10.10.10.1 7      # Echo server
telnet 10.10.10.1 13     # Daytime server
telnet 10.10.10.1 80     # HTTP — then type:
                         # GET / HTTP/1.1
                         # Host: anything
                         # (press Enter twice)
telnet 10.10.10.1 25     # SMTP
telnet 10.10.10.1 110    # POP3
telnet 10.10.10.1 143    # IMAP
```
`Ctrl + ]` to escape telnet session, then type `quit`

---

## TLS / SSL & Secure Protocols

**SSL** → developed by Netscape 1995 (now deprecated)  
**TLS** → IETF standard 1999, current version TLS 1.3 (2018)

TLS adds **confidentiality, integrity, and authenticity** to existing protocols without modifying the underlying IP or TCP layers.

### How TLS Works (simplified)
1. Server gets a signed certificate from a **Certificate Authority (CA)**
2. Client verifies certificate against trusted CAs installed on the system
3. TLS handshake negotiates encryption — all subsequent traffic is encrypted

**Self-signed certificates** can't prove server authenticity — no third party verified them.

### HTTPS Connection Flow
1. TCP three-way handshake
2. TLS negotiation/establishment (several packets)
3. Encrypted HTTP data exchange

Traffic appears as "Application Data" in Wireshark — can't be read without the key.

---

## SSH

Replaced TELNET for secure remote access. Developed 1995, OpenSSH released 1999.  
Listens on **TCP port 22**.

```bash
ssh username@hostname
ssh username@hostname -X    # X11 forwarding (GUI apps over SSH)
```

**Features over TELNET:**
- Encrypted (end-to-end)
- Public key + 2FA authentication options
- Man-in-the-middle protection (alerts on new/changed server keys)
- Tunneling (route other protocols through SSH)
- X11 forwarding

---

## VPN

Routes traffic through an encrypted tunnel over the public internet.  
**Use cases:** Connect remote offices to main branch, secure remote workers, bypass geo-restrictions  

When connected, external services see the **VPN server's IP**, not yours.  
ISP only sees encrypted traffic.

**Note:** VPN usage is illegal in some countries — check local laws.

---

## Notes / Things to Add

*Use this space as you pick up new stuff:*

-
-
-
