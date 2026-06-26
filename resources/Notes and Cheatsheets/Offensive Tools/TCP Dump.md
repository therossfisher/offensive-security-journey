# Tcpdump — Study Notes

---

## What Is Tcpdump?

Command-line packet capture and analysis tool. Written in C/C++, released for Unix-like systems late 1980s/early 1990s. Built on the **libpcap** library — the same foundation used by Wireshark and many other tools.

Ported to Windows as **winpcap** (and its successor **npcap**, which Wireshark uses on Windows).

**Key difference from Wireshark:** No GUI — entirely command-line. Faster, more scriptable, and available on nearly every Linux/Unix system by default. Essential for working on headless servers where a GUI isn't available.

---

## Core Command Flags

### Capture Control

| Flag | Purpose |
|------|---------|
| `-i INTERFACE` | Specify interface to listen on |
| `-i any` | Listen on all interfaces |
| `-c COUNT` | Stop after capturing COUNT packets |
| `-w FILE.pcap` | Write packets to file (no output to screen) |
| `-r FILE.pcap` | Read from a saved capture file |

### Output Control

| Flag | Purpose |
|------|---------|
| `-n` | Don't resolve IP addresses to hostnames |
| `-nn` | Don't resolve IPs **or** port numbers |
| `-v` | Verbose output (TTL, ID, length, options) |
| `-vv` | More verbose |
| `-vvv` | Maximum verbosity |
| `-q` | Quick/quiet — brief one-line output |
| `-e` | Include MAC addresses (link-layer header) |
| `-A` | Print payload as ASCII |
| `-xx` | Print payload as hex (including headers) |
| `-X` | Print payload as **both** hex and ASCII |

### Check available interfaces first:
```bash
ip a s
```

---

## Basic Examples

```bash
# Capture 50 packets on eth0 verbosely
sudo tcpdump -i eth0 -c 50 -v

# Capture on WiFi, save to file, run until stopped with Ctrl+C
sudo tcpdump -i wlo1 -w capture.pcap

# Read a saved file, show 5 packets, no DNS resolution
tcpdump -r capture.pcap -c 5 -n

# Capture on all interfaces, no name resolution
sudo tcpdump -i any -nn

# SSH traffic only
sudo tcpdump -i any tcp port 22
```

---

## Filtering (BPF — Berkeley Packet Filter)

Tcpdump uses **BPF syntax** for filters. Same filter language used by Wireshark's capture filters and many other tools.

### By Host

```bash
tcpdump host 192.168.1.1          # Traffic to OR from this IP
tcpdump src host 192.168.1.1      # Traffic FROM this IP only
tcpdump dst host 192.168.1.1      # Traffic TO this IP only
tcpdump host example.com          # Resolves hostname
```

### By Port

```bash
tcpdump port 53                   # DNS (UDP + TCP)
tcpdump port 80                   # HTTP
tcpdump src port 443              # Traffic sourced from port 443
tcpdump dst port 22               # Traffic destined to SSH
```

### By Protocol

```bash
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp
tcpdump ip
tcpdump ip6
```

### Logical Operators

```bash
tcpdump host 1.1.1.1 and tcp           # Both conditions must match
tcpdump udp or icmp                    # Either condition matches
tcpdump not tcp                        # Everything except TCP
```

### Size Filters

```bash
tcpdump greater 1000      # Packets >= 1000 bytes
tcpdump less 100          # Packets <= 100 bytes
```

### Practical Examples

```bash
# All SSH traffic
sudo tcpdump -i any tcp port 22

# NTP traffic on WiFi
sudo tcpdump -i wlo1 udp port 123

# HTTPS traffic to/from a specific host, saved to file
sudo tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap

# Count packets matching a filter (pipe to wc)
tcpdump -r capture.pcap src host 192.168.1.1 -n | wc

# All non-SSH traffic
sudo tcpdump -i any not port 22
```

---

## Advanced Filtering — TCP Flags

TCP flags are stored in a single byte in the TCP header. Tcpdump lets you filter on them using `tcp[tcpflags]`.

### Available Flag Names

| Flag | Meaning |
|------|---------|
| `tcp-syn` | SYN — connection initiation |
| `tcp-ack` | ACK — acknowledgment |
| `tcp-fin` | FIN — connection teardown |
| `tcp-rst` | RST — connection reset (abrupt close) |
| `tcp-push` | PSH — push data to app immediately |

### Flag Filter Syntax

```bash
# ONLY the SYN flag set (nothing else) — new connection attempts
tcpdump "tcp[tcpflags] == tcp-syn"

# SYN flag set (may have others too) — broader catch
tcpdump "tcp[tcpflags] & tcp-syn != 0"

# SYN OR ACK set — catch handshakes
tcpdump "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"

# Only RST flag set — abrupt connection resets
tcpdump "tcp[tcpflags] == tcp-rst"

# FIN flag — connection teardowns
tcpdump "tcp[tcpflags] & tcp-fin != 0"
```

**Why RST packets matter for security:** A flood of RST packets can indicate a port scan (scanners send RST after getting a response), a connection being forcibly terminated, or a firewall dropping connections. Worth filtering for in incident response.

**Why SYN-only packets matter:** A high volume of SYN packets with no corresponding SYN-ACK responses is a classic indicator of a **SYN flood DoS attack**.

---

## Header Byte Filtering (Advanced)

Tcpdump can filter on any byte of any protocol header using `proto[offset:size]` syntax:

```
proto[expr:size]

proto  = arp, ether, icmp, ip, ip6, tcp, udp
expr   = byte offset (0 = first byte)
size   = number of bytes to read (default 1)
```

**Examples:**
```bash
# Packets to a multicast Ethernet address
tcpdump "ether[0] & 1 != 0"

# IP packets with options set
tcpdump "ip[0] & 0xf != 5"
```

You won't need this daily, but it shows up in advanced threat hunting and malware traffic analysis. Understanding it helps you read other people's complex filters.

---

## Output Format Tips

### When to use which output format

| Scenario | Best flag |
|----------|-----------|
| Quick triage, just need IPs and ports | `-q` |
| Need to see MAC addresses (ARP, DHCP analysis) | `-e` |
| Reading plaintext protocol data (HTTP, FTP, SMTP) | `-A` |
| Encrypted traffic or binary data inspection | `-X` (hex + ASCII) |
| Deep header inspection | `-xx` |
| Writing a script that parses output | `-nn -q` |

---

## Reading Output — Packet Line Breakdown

Standard tcpdump output line:
```
18:55:18.989213 IP 10.10.1.1.22 > 10.11.1.2.54321: Flags [P.], seq 100:200, ack 1, win 922, length 196
```

| Field | Meaning |
|-------|---------|
| `18:55:18.989213` | Timestamp |
| `IP` | Network protocol |
| `10.10.1.1.22` | Source IP.port |
| `10.11.1.2.54321` | Destination IP.port |
| `Flags [P.]` | TCP flags (P = PSH, . = ACK) |
| `seq 100:200` | Sequence number range |
| `ack 1` | Acknowledgment number |
| `win 922` | Window size |
| `length 196` | Payload length in bytes |

### TCP Flag Abbreviations in Output

| Symbol | Flag |
|--------|------|
| `S` | SYN |
| `.` | ACK |
| `P` | PSH |
| `F` | FIN |
| `R` | RST |
| `U` | URG |
| `[S.]` | SYN-ACK |
| `[P.]` | PSH+ACK (data transfer) |
| `[F.]` | FIN+ACK (teardown) |

---

## Useful One-Liners (Beyond the Room)

```bash
# Save capture for 60 seconds then stop
sudo tcpdump -i eth0 -w capture.pcap & sleep 60; kill %1

# Capture and immediately search for cleartext passwords
sudo tcpdump -i any -A -nn port 21 or port 23 or port 110 | grep -i "pass"

# Watch for new hosts ARP-ing onto a network
sudo tcpdump -i eth0 arp -e -n

# Capture DNS queries only (no responses)
sudo tcpdump -i any udp port 53 and src not 192.168.1.1

# Look for potential SYN scan activity (SYN only, no established connections)
sudo tcpdump -i any "tcp[tcpflags] == tcp-syn" -nn

# Rotate capture files every 100MB, keep 5 files
sudo tcpdump -i eth0 -w capture-%Y%m%d%H%M%S.pcap -C 100 -W 5

# Read a file and output to Wireshark in real time (over SSH)
ssh user@remote "sudo tcpdump -i eth0 -w - not port 22" | wireshark -k -i -
```

The last one is particularly useful — pipe a remote tcpdump capture directly into a local Wireshark session over SSH. Lets you use Wireshark's GUI to analyze traffic on a headless server.

---

## Tcpdump vs Wireshark — When to Use Which

| Situation | Tool |
|-----------|------|
| Headless server, no GUI | Tcpdump |
| Quick triage on live traffic | Tcpdump |
| Scripting/automation | Tcpdump |
| Deep packet analysis, stream following | Wireshark |
| Decrypting TLS with key logs | Wireshark |
| File extraction from capture | Wireshark |
| Sharing findings with a team | Wireshark (visual) |

**Best practice:** Capture with tcpdump, analyze with Wireshark. They read the same .pcap format.

---

## For Your GIAC Path

The **GSEC** and **GCIH** exams both cover packet analysis. Things to understand well:

- Reading TCP flag patterns to identify scan types (SYN scan = SYN only, no ACK back; XMAS scan = FIN+PSH+URG; NULL scan = no flags)
- Recognizing three-way handshake vs RST vs half-open connections in output
- Using tcpdump to isolate traffic around a specific incident timeframe
- Correlating tcpdump output with Suricata alert timestamps from your homelab

Your Suricata + eve.json setup pairs naturally with tcpdump — when Suricata fires an alert, you can pull the exact packets around that timestamp from a parallel tcpdump capture to see what triggered it.

---

## Notes / Things to Add

*Use this space as you go deeper:*

-
-
-

---

## Addendum — Lab Notes (Tcpdump: The Basics Room)

*Things I worked through hands-on while completing the room.*

---

### Finding a Specific ARP Request

Simplest way — matches both sender and target:
```bash
tcpdump -i eth0 arp and host 192.168.1.10
```

If you need ONLY ARP requests (not replies) for a specific IP using byte offsets:
```bash
tcpdump -i eth0 'arp[6:2] = 1 and arp[24:4] = 0xc0a8010a'
```
- `arp[6:2] = 1` → opcode 1 = ARP Request
- `arp[24:4]` → target IP field in the ARP payload
- Convert IP to hex: `printf '%02x%02x%02x%02x\n' 192 168 1 10`

---

### Finding the First DNS Query in a PCAP

```bash
tcpdump -nn -r capture.pcap udp port 53
```
First line of output = first DNS packet. Use `-vv` to see the domain name clearly.

To filter ONLY queries (not responses) using the DNS QR bit:
```bash
tcpdump -nn -r capture.pcap 'udp port 53 and udp[10] & 0x80 = 0'
```
- `udp[10] & 0x80 = 0` → QR bit = 0 means query; QR bit = 1 means response

---

### Finding the Host That Sent Packets Larger Than X Bytes

```bash
tcpdump -nn -r traffic.pcap 'greater 15000'
```

To extract and count source IPs:
```bash
tcpdump -nn -r traffic.pcap 'greater 15000' \
  | awk '{print $3}' \
  | cut -d. -f1-4 \
  | sort | uniq -c | sort -nr
```

To just grab the first matching host quickly:
```bash
tcpdump -nn -r traffic.pcap 'greater 15000' \
  | head -1 \
  | awk '{print $3}' \
  | cut -d. -f1-4
```

Alternative using IP header byte offset (more precise — checks IP total length field):
```bash
tcpdump -nn -r traffic.pcap 'ip[2:2] > 15000'
```

---

### Getting MAC Addresses from ARP — The `-e` Flag Gotcha

By default, tcpdump doesn't print Ethernet headers. This output:
```
07:18:29.940761 ARP, Request who-has 192.168.124.137 tell 192.168.124.148, length 28
```
...has no MAC address visible. Fix it with `-e`:

```bash
tcpdump -nn -e -r traffic.pcap arp
```

This forces the link-layer (Ethernet) header to print, giving you:
```
52:54:00:aa:bb:cc > ff:ff:ff:ff:ff:ff, ARP, Request who-has ...
```

**Important gotcha:** If the PCAP was captured with a small `snaplen`, the Ethernet header may have been truncated and the MAC won't appear even with `-e`. In that case:
- **ARP Reply** → MAC IS recoverable (it's inside the ARP payload: `is-at xx:xx:xx:xx:xx:xx`)
- **ARP Request** → MAC may NOT be recoverable (it only lived in the Ethernet header that got cut off)

Check snaplen with:
```bash
tcpdump -r traffic.pcap -vvv | head
```

---

### Counting Packets Matching a Filter (Pipe to `wc`)

```bash
tcpdump -r traffic.pcap "tcp[tcpflags] == tcp-rst" | wc
```
The first number in `wc` output = number of lines = number of matching packets.

Add `-nn` to speed it up and avoid DNS delays:
```bash
tcpdump -nn -r traffic.pcap "tcp[tcpflags] == tcp-rst" | wc -l
```
`wc -l` counts lines only — cleaner output than bare `wc`.
