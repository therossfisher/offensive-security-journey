# Nmap — Professional Cheat Sheet & Study Notes

---

## Syntax Reference

```bash
sudo nmap [scan type] [options] [target]

# Standard starting point for most recon
sudo nmap -sC -sV --reason -oA nmap/initial TARGET_IP
```

---

## Target Specification

| Format | Example | Meaning |
|--------|---------|---------|
| Single IP | `192.168.1.1` | One host |
| Range | `192.168.1.1-10` | 192.168.1.1 through .10 |
| Subnet | `192.168.1.0/24` | All 256 addresses |
| Hostname | `example.thm` | DNS-resolved target |
| List file | `-iL targets.txt` | Read targets from file |
| Exclude | `--exclude 192.168.1.1` | Skip specific IPs |
| Exclude file | `--excludefile skip.txt` | Skip IPs in file |

---

## Host Discovery

| Flag | What It Does |
|------|-------------|
| `-sn` | Ping scan only — discover live hosts, no port scan |
| `-sL` | List scan — prints targets without scanning them (safe recon) |
| `-Pn` | Skip host discovery — treat all hosts as online |
| `-PS[ports]` | TCP SYN ping to specific ports |
| `-PA[ports]` | TCP ACK ping to specific ports |
| `-PU[ports]` | UDP ping to specific ports |
| `-PP` | ICMP timestamp ping (when echo is blocked) |
| `-PE` | ICMP echo ping (default) |
| `-n` | No DNS resolution (faster) |
| `-R` | Force reverse DNS on all hosts |
| `--dns-servers IP` | Use specific DNS server |

**Local vs Remote behavior:**
- Same subnet: Nmap uses **ARP requests** first (faster, always works on LAN)
- Across routers: Nmap uses **ICMP, TCP SYN to 443, TCP ACK to 80**

**When ICMP is blocked — escalation path:**
1. `-PP` — ICMP timestamp
2. `-PU` — UDP ping (triggers ICMP port unreachable from closed ports)
3. `-PA` — TCP ACK ping (requires root)
4. `-Pn` — skip discovery entirely, scan anyway

---

## Port Scanning

### Scan Types 

| Flag | Name | How It Works | Noise Level |
|------|------|-------------|-------------|
| `-sT` | Connect Scan | Full 3-way handshake — logged by target | High |
| `-sS` | SYN Scan (Stealth) | Half-open — sends SYN, RSTs on reply | Medium |
| `-sU` | UDP Scan | Sends UDP probes — slow, often skipped | Low |
| `-sN` | Null Scan | No flags set — closed ports reply RST | Low |
| `-sF` | FIN Scan | FIN flag only — closed ports reply RST | Low |
| `-sX` | Xmas Scan | FIN+PSH+URG — closed ports reply RST | Low |
| `-sM` | Maimon Scan | FIN+ACK — BSD systems drop if port open | Low |
| `-sA` | ACK Scan | Maps firewall rules, doesn't find open ports | Low |
| `-sW` | Window Scan | ACK + window field analysis | Low |
| `-sI ZOMBIE` | Idle Scan | Uses a zombie host — your IP never touches target | Lowest |
| `--scanflags FLAGS` | Custom Scan | Set any TCP flag combination manually | Varies |

**Root required:** `-sS`, `-sU`, `-sN`, `-sF`, `-sX`, `-sM`, `-sA`, `-sW`, `-sI`  
**No root needed:** `-sT` (but it's noisy and fully logged)  
**Key note:** `-sV` forces a full TCP handshake — stealth SYN is not possible with version detection active.

### How Stealthy Scans Work
Null, FIN, and Xmas scans rely on RFC behavior:
- **Closed port** → responds with RST (port is closed)
- **Open port** → no response (silence = possibly open)
- **Filtered port** → no response (same as open — can't tell the difference)

Doesn't work reliably against Windows (Microsoft ignores the RFC on this).

### ACK Scan Purpose
`-sA` doesn't find open ports — it reveals **firewall rules**:
- **Unfiltered** response = packet reached the host (firewall allows it)
- **No response** = firewall is blocking it
Use it to map what a firewall is and isn't blocking.

### Idle (Zombie) Scan
```bash
nmap -sI ZOMBIE_IP TARGET_IP
```
Your IP never touches the target. All probes appear to come from the zombie.

**How it works:**
1. Check zombie's IP ID value
2. Send SYN to target spoofed as zombie's IP
3. Check zombie's IP ID again

| IP ID Change | Meaning |
|-------------|---------|
| +1 | Port closed/filtered — target sent RST to zombie, zombie ignored it |
| +2 | Port **open** — target sent SYN-ACK to zombie, zombie RST'd back (increments IP ID) |

Good zombie candidates: idle printers, IoT devices, unused workstations

---

## Port Selection

| Flag | Description |
|------|-------------|
| `-p22,80,443` | Specific ports |
| `-p1-1023` | Port range |
| `-p-` | All 65535 ports |
| `-F` | Fast — top 100 ports |
| `--top-ports 10` | Top N most common ports |
| `-r` | Scan ports in consecutive order (not random) |

**Well-known ports:** 1–1024. Start here if `-F` misses something interesting.  
**Full scan tip:** Run `-F` first for speed, then `-p-` to catch non-standard services.

---

## Port States

| State | Meaning |
|-------|---------|
| Open | A service is actively listening |
| Closed | Port accessible but no service listening |
| Filtered | Nmap can't determine state — firewall/packet filter in the way |
| Unfiltered | Port is accessible but open/closed can't be determined (ACK scan result) |
| Open\|Filtered | Can't tell if open or filtered (Null/FIN/Xmas/UDP) |
| Closed\|Filtered | Can't tell if closed or filtered (Idle scan only) |

---

## TCP Flags Reference

| Flag | Meaning | Common Use |
|------|---------|-----------|
| SYN | Initiate connection | Start of handshake |
| ACK | Acknowledge received data | Part of every established connection |
| FIN | No more data to send | Clean connection close |
| RST | Reset / forcibly close | Rejected connection, port closed |
| PSH | Push data to application immediately | Minimize buffering delay |
| URG | Process this data urgently | Rare in modern traffic |

---

## Service & Version Detection

| Flag | Description |
|------|-------------|
| `-sV` | Detect service name and version |
| `-sV --version-light` | Intensity 2 — faster, less accurate |
| `-sV --version-all` | Intensity 9 — most thorough |
| `--version-intensity [0-9]` | Manual intensity control |
| `-O` | OS detection via TCP/IP fingerprinting |
| `-A` | Aggressive: `-sV -sC -O --traceroute` |
| `--traceroute` | Map hops to target |

**TTL OS hints (when `-O` is uncertain):**

| TTL | Likely OS |
|-----|----------|
| 64 | Linux / Unix |
| 128 | Windows |
| 255 | Network device / Cisco |

**Note:** Virtualisation, cloud environments, and firewalls can alter TTL values.

---

## NSE — Nmap Scripting Engine

Scripts live at: `/usr/share/nmap/scripts/`  
Named by protocol: `http-*`, `ftp-*`, `smb-*`, `ssh-*`, `vuln-*`, etc.

| Flag | Description |
|------|-------------|
| `-sC` or `--script=default` | Run default scripts |
| `--script=NAME` | Run specific script |
| `--script="ftp*"` | All scripts matching pattern |
| `--script-args KEY=VALUE` | Pass arguments to scripts |
| `--script-help NAME` | Show script documentation |

### Script Categories

| Category | Description | Noise Level |
|----------|-------------|-------------|
| `auth` | Authentication bypass checks | Low |
| `brute` | Password brute force | High |
| `default` | Safe, commonly useful — runs with `-sC` | Low |
| `discovery` | Enumerate DNS, databases, SNMP, etc. | Medium |
| `exploit` | Attempt to exploit vulnerabilities | High |
| `external` | Queries external resources | Low |
| `fuzzer` | Send unexpected inputs to find bugs | High |
| `intrusive` | Aggressive — may crash services | High |
| `malware` | Check for backdoors | Low |
| `safe` | Won't crash target — low impact | Low |
| `version` | Service version detection | Low |
| `vuln` | Check for known CVEs | Medium |

**Useful scripts to know:**

| Script | Purpose |
|--------|---------|
| `http-title` | Get webpage title |
| `http-robots.txt` | Check robots.txt |
| `smb-vuln-ms17-010` | EternalBlue check |
| `smb-enum-shares` | List SMB shares |
| `ftp-anon` | Check for anonymous FTP login |
| `ssh-hostkey` | Get SSH host keys |
| `dns-brute` | Subdomain enumeration |
| `vuln` category | Broad CVE check (noisy) |

```bash
# Run specific script
nmap --script=http-title TARGET_IP

# Run multiple scripts
nmap --script=smb-vuln-ms17-010,smb-enum-shares TARGET_IP

# Run all SMB scripts
nmap --script="smb*" TARGET_IP
```

---

## Timing & Performance

### Timing Templates

| Flag | Name | Delay Between Probes | Use Case |
|------|------|---------------------|---------|
| `-T0` | Paranoid | 5 minutes | Maximum IDS evasion |
| `-T1` | Sneaky | 15 seconds | Real engagements |
| `-T2` | Polite | 0.4 seconds | Reducing network impact |
| `-T3` | Normal | Default (adaptive) | Default |
| `-T4` | Aggressive | Minimal | CTFs and lab environments |
| `-T5` | Insane | Fastest possible | May miss results |

### Fine-Grained Control

| Flag | Description |
|------|-------------|
| `--min-rate N` | Minimum N packets/second across whole scan |
| `--max-rate N` | Maximum N packets/second |
| `--min-parallelism N` | Minimum parallel probes |
| `--max-parallelism N` | Maximum parallel probes |
| `--host-timeout TIME` | Give up on a host after this long |
| `--scan-delay TIME` | Minimum delay between probes to one host |
| `--max-scan-delay TIME` | Maximum delay between probes |

```bash
# Cap rate to look less like a scan
sudo nmap -sS --max-rate 10 TARGET_IP

# Force high parallelism for speed
sudo nmap -sS --min-parallelism 512 TARGET_IP
```

---

## Firewall & IDS Evasion

### Packet Fragmentation

| Flag | Fragment Size | TCP Header Fragments |
|------|--------------|---------------------|
| `-f` | 8 bytes | 3 fragments (24 ÷ 8) |
| `-ff` | 16 bytes | 2 fragments (16 + 8) |
| `--mtu N` | Custom (multiple of 8) | Varies |
| `--data-length N` | Pads packets with random data | Makes packets look normal-sized |

```bash
sudo nmap -sS -f TARGET_IP           # 8-byte fragments
sudo nmap -sS -ff TARGET_IP          # 16-byte fragments
sudo nmap -sS --mtu 24 TARGET_IP     # Custom fragment size
sudo nmap -sS --data-length 200 TARGET_IP  # Pad to 200 bytes
```

### Spoofing & Decoys

```bash
# Spoof source IP (only useful if you can capture responses)
sudo nmap -e eth0 -Pn -S SPOOFED_IP TARGET_IP

# Spoof source MAC (same subnet only)
sudo nmap --spoof-mac SPOOFED_MAC TARGET_IP

# Decoy scan — your IP hidden among fake IPs
sudo nmap -D 10.10.0.1,10.10.0.2,ME TARGET_IP
sudo nmap -D 10.10.0.1,RND,RND,ME TARGET_IP

# Spoof source port (bypass some firewall rules)
sudo nmap --source-port 53 TARGET_IP
```

| Value | Meaning |
|-------|---------|
| `ME` | Your real IP — controls its position in decoy list |
| `RND` | Random IP generated at scan time |

**Source port spoofing:** Some firewalls allow traffic from "trusted" ports like 53 (DNS) or 80. Using `--source-port 53` can bypass these rules.

---

## Output & Verbosity

### Save Formats

| Flag | Format | Best For |
|------|--------|---------|
| `-oN scan.nmap` | Normal | Human reading |
| `-oG scan.gnmap` | Grepable | Scripting, bulk grep |
| `-oX scan.xml` | XML | Tool integration (Metasploit, etc.) |
| `-oA scan` | All three | Real engagements — always use this |
| `-oS scan.txt` | Script kiddie | Useless novelty |

```bash
# Always save — you never know what you'll need
sudo nmap -sC -sV -oA nmap/initial TARGET_IP

# Import into Metasploit
msf> db_import /path/to/scan.xml
# Or scan directly from msfconsole
msf> db_nmap -sV -sC TARGET_IP
```

### Verbosity & Debugging

| Flag | Description |
|------|-------------|
| `-v` | Verbose — see results as they come in |
| `-vv` or `-v2` | More verbose |
| `-vvvv` or `-v4` | Maximum verbosity |
| `-d` | Debugging output |
| `-d9` | Maximum debugging (thousands of lines) |
| `--reason` | Shows *why* Nmap classified each port |
| `--open` | Only show open ports (cleaner output) |
| `--packet-trace` | Show all packets sent and received |

**`--reason` is underused** — when a port shows filtered or a result looks wrong, `--reason` tells you exactly what response (or lack of response) Nmap got. Saves a lot of troubleshooting time.

---

## Common Scan Workflows

### Initial Recon (Standard Starting Point)
```bash
# Quick check — what's alive and what's obvious
sudo nmap -sC -sV --reason -oA nmap/initial TARGET_IP

# Follow up with full port scan
sudo nmap -p- -sV -oA nmap/full TARGET_IP
```

### Lab / CTF Workflow
```bash
# Fast and thorough
sudo nmap -A -T4 -p- -oA nmap/ctf TARGET_IP
```

### Stealth Recon (Real Engagement)
```bash
# Slow, fragmented, decoys
sudo nmap -sS -T1 -f -D RND,RND,ME -oA nmap/stealth TARGET_IP
```

### Subnet Host Discovery
```bash
# Find live hosts on a /24
sudo nmap -sn 192.168.1.0/24

# Save discovered hosts for follow-up
sudo nmap -sn 192.168.1.0/24 -oG hosts.gnmap
grep "Up" hosts.gnmap | awk '{print $2}' > live_hosts.txt
sudo nmap -iL live_hosts.txt -sV -oA nmap/services
```

### Service-Specific Scans
```bash
# Web recon
sudo nmap -sV -p80,443,8080,8443 --script="http*" TARGET_IP

# SMB check (Windows)
sudo nmap -p445 --script="smb-vuln*,smb-enum*" TARGET_IP

# FTP check
sudo nmap -p21 --script=ftp-anon,ftp-bounce TARGET_IP

# Full UDP scan (slow — takes a long time)
sudo nmap -sU -T4 --top-ports 100 TARGET_IP
```

### Grep Tricks on Grepable Output
```bash
# Find all hosts with port 80 open across many scan files
grep "80/open" scan.gnmap

# Extract just IPs from a ping scan
grep "Up" hosts.gnmap | cut -d " " -f 2

# Find all open ports across the scan
grep "open" scan.gnmap
```

---

## OS Detection Notes

Nmap's OS detection (`-O`) uses TCP/IP stack fingerprinting — window size, TTL, TCP options, and other behaviors differ between implementations. Not always accurate. Support with:

- TTL analysis from ping/traceroute output
- Banner grabbing (`-sV` often reveals OS in service banners)
- HTTP headers (`Server:` field)
- SMB fingerprinting (reveals Windows version)

---

## Privilege Notes

| Privilege Level | Scan Available |
|----------------|----------------|
| Root / sudo | All scan types |
| Local user | Connect scan (`-sT`) only — no raw packet crafting |

**Default behavior:**
- With sudo → `-sS` (SYN scan) automatically
- Without sudo → `-sT` (connect scan) automatically

Always run with sudo in practice. The difference in scan quality is significant.

---

## Nmap + Metasploit Integration

```bash
# Inside msfconsole — scan and auto-import to database
msf> db_nmap -sV -sC TARGET_IP

# Import existing XML output
msf> db_import /path/to/scan.xml

# Query the database after scanning
msf> hosts
msf> services
msf> vulns
```

---

## For Your GIAC Path (GSEC/GCIH)

Things that appear in exam scenarios:

- **Recognizing scan type from packet capture** — SYN scan leaves half-open connections (SYN → SYN-ACK → RST). Connect scan completes the handshake then FIN. Null/FIN/Xmas = unusual flag combos in Wireshark.
- **Interpreting port states** — `filtered` means firewall. `closed` means host is up but no service. `open|filtered` = Null/FIN/Xmas on an open port.
- **ACK scan for firewall mapping** — not for finding services, for finding firewall rules.
- **Timing in real engagements** — T1/T2 with `--max-rate` and fragmentation together for evasion. Never T4/T5 on a real target.
- **Why UDP is often skipped and shouldn't be** — DNS, SNMP, NTP, DHCP, TFTP all run UDP. Attackers exploit these because defenders don't scan UDP.

Your Suricata setup will alert on Nmap scans using default timing. Run a SYN scan against your own homelab and watch the `et:scan` rule category fire — it's a good way to understand what signatures detect.

---

## Notes / Things to Add

*Use this space as you go deeper:*

-
-
-
