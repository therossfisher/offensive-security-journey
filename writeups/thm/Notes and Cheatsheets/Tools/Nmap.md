# Nmap Cheat Sheet

## Syntax
```
sudo nmap -sC -sV --reason -oA nmap/facts
```

---

## Common Scans

| Command | Description |
|---|---|
| `nmap 10.10.10.5` | Basic scan — top 1000 ports |
| `nmap -p- 10.10.10.5` | Scan all 65535 ports |
| `nmap -p 22,80,445 10.10.10.5` | Scan specific ports |
| `nmap -p 1-1000 10.10.10.5` | Scan port range |

---

## Key Switches

| Switch | Description |
|---|---|
| `-sV` | Service/version detection |
| `-sC` | Run default scripts (adds extra recon) |
| `-sS` | SYN scan — stealthy, doesn't complete handshake (requires root) |
| `-sU` | UDP scan |
| `-sn` | Ping sweep — host discovery only, no port scan |
| `-O` | OS detection |
| `-A` | Aggressive — enables `-sV -sC -O` and traceroute |
| `-T4` | Faster scan speed (T0-T5, T4 is common for labs) |
| `-v` | Verbose output |
| `-p-` | Scan all ports |

---

## Output

| Switch | Description |
|---|---|
| `-oN scan.txt` | Save output to file (normal format) |
| `-oX scan.xml` | Save as XML |
| `-oG scan.grep` | Save as greppable format |
| `-oA scan` | Save all three formats at once |

---

## Common Combos

```bash
# Standard recon scan
nmap -sV -sC -oN scan.txt 10.10.10.5

# Full port scan with version detection
nmap -sV -p- -oN fullscan.txt 10.10.10.5

# Fast host discovery on a subnet
nmap -sn 10.10.10.0/24

# Aggressive scan
nmap -A -oN aggressive.txt 10.10.10.5
```

---
## Reverse DNS Lookup

**What it is:** Normal DNS resolves a hostname to an IP. Reverse DNS goes the other way — given an IP, it asks "what hostname is associated with this?"

**Why it matters in pentesting:** Hostnames reveal information. An IP that resolves to `dev-internal.corp.com` or `db-server.corp.com` tells you what that machine does without scanning it.

**Nmap behavior:**
- Nmap performs reverse DNS lookup by default on discovered hosts
- The hostname shows up in scan output next to the IP

**Relevant flags:**

| Flag                 | Description                                                     |
| -------------------- | --------------------------------------------------------------- |
| `-n`                 | Disable reverse DNS lookup — faster scans, less noise           |
| `-R`                 | Force reverse DNS lookup on all hosts, even offline ones        |
| `--dns-servers [IP]` | Use a specific DNS server for lookups instead of system default |

**When to use `-n`:** When scanning large ranges and speed matters more than hostname info. Common in the early wide-net phase of recon.
## Notes
- Always save output with `-oN` — you'll want to reference it later
- Run full port scan (`-p-`) when initial scan comes back with nothing interesting
- Use inside msfconsole with `db_nmap` to auto-save results to database

## Nmap port states

1. Open - a service is listening on the port
2. Closed - no service is listening although the port is accessible, i.e.- not blocked by firewall or other appliance/program. 
3. Filtered - Nmap cannot determine whether port is open or closed because it is not accessible. 
4. Unfiltered - Nmap cannot determine whether port is open or closed even though it is accessible, encountered when using an ACK scan ```-sA```. 
5. Open|Filtered - Nmap can't determine if the port is open or filtered
6. Closed|Filtered - Nmap can't decide if port is closed or filtered

## Nmap TCP Flags that can be set

1. URG: indicates to process immediately without waiting for previously sent TCP segments
2. ACK: acknowledges receipt of a TCP segment
3. PSH: asking TCP to pass the data to the app. promptly
4. RST: used to reset or tear down connections, also sent to a host when there is no service on the receiving end to answer
5. SYN: initiates a TCP 3-way handshake and synchronize sequence numbers between hosts. 
6. FIN: No more data to send

## Scan Types

| Flag        | Name            | Description                                                                                                                                                   |
| ----------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-sT`       | Connect Scan    | Completes full TCP handshake — no root required, but noisy and easily logged                                                                                  |
| `-sS`       | SYN Scan        | Half-open scan — sends SYN, doesn't complete handshake — stealthy, requires root                                                                              |
| `-sU`       | UDP Scan        | Scans UDP ports — slow, often missed — requires root                                                                                                          |
| -sN         | Null scan       | NO flags set                                                                                                                                                  |
| -sF         | FIN Scan        | Send packets with FIN flag set                                                                                                                                |
| -sX         | Xmas Scan       | Send packets with FIN, PSH and URG flags simlutaneously                                                                                                       |
| -sM         | Maimon Scan     | Send packets with FIN and ACK set, some BSD systems drop packet if port is open, exposing it.                                                                 |
| -sA         | ACK Scan        | Send packets with ACK flag set                                                                                                                                |
| -sW         | TCP Window Scan | Similar to ACK scan but examines the TCP window field of the RST packets returned.                                                                            |
| --scanflags | Custom scan     | Can set custom scan with ```--scanflags RSTSYNFIN``` for example, but need to understand how different ports will behave to be able to interpret the results. |

## Port Order
| Flag | Description |
|---|---|
| `-r` | Scan ports in consecutive order instead of random order |
## Fine-Tuning Scans

### Port Selection

| Flag | Description |
|---|---|
| `-p22,80,443` | Scan specific ports |
| `-p1-1023` | Scan a port range |
| `-p-` | Scan all 65535 ports |
| `-F` | Fast mode — top 100 ports only |
| `--top-ports 10` | Scan the 10 most common ports |

---

### Timing Templates

| Flag | Name | Use Case |
|---|---|---|
| `-T0` | Paranoid | Maximum stealth — 5 min between probes |
| `-T1` | Sneaky | Real engagements where IDS evasion matters |
| `-T2` | Polite | Reduces bandwidth usage |
| `-T3` | Normal | Default |
| `-T4` | Aggressive | CTFs and practice labs |
| `-T5` | Insane | Fastest — may drop packets and miss results |

---

### Rate and Parallelism Control

| Flag | Description |
|---|---|
| `--max-rate 10` | Cap packets per second (e.g. no more than 10/sec) |
| `--min-rate 100` | Floor packets per second |
| `--min-parallelism 512` | Minimum parallel probes running at once |
| `--max-parallelism 512` | Maximum parallel probes running at once |
## Spoofing and Decoys

### IP Spoofing
Makes scan packets appear to come from a different IP address. Only useful if you can capture the responses (e.g. you're monitoring the network traffic).

```bash
# Basic spoof — rarely useful alone
nmap -S SPOOFED_IP TARGET_IP

# Practical spoof — specify interface and disable ping
nmap -e eth0 -Pn -S SPOOFED_IP TARGET_IP
```

**How it works:**
1. You send packets with a fake source IP
2. Target responds to the fake IP, not you
3. You must be monitoring the network to capture those replies

---

### MAC Spoofing
Only works if you're on the same subnet (same Ethernet or WiFi network) as the target.

```bash
nmap --spoof-mac SPOOFED_MAC TARGET_IP
```

---

### Decoy Scans
Makes the scan appear to come from multiple IPs so your real IP is harder to identify. More practical than IP spoofing because you still get the responses.

```bash
# Specific decoys with your IP in third position
nmap -D 10.10.0.1,10.10.0.2,ME TARGET_IP

# Mix of specific, random, and your IP
nmap -D 10.10.0.1,10.10.0.2,RND,RND,ME TARGET_IP
```

| Value | Description |
|---|---|
| `10.10.0.1` | Specific decoy IP |
| `RND` | Random IP generated each run |
| `ME` | Your real IP — controls where in the list it appears |

---

### Key Flags

| Flag | Description |
|---|---|
| `-S SPOOFED_IP` | Spoof source IP address |
| `-e eth0` | Specify network interface to use |
| `--spoof-mac MAC` | Spoof source MAC address (same subnet only) |
| `-D decoy1,decoy2,ME` | Launch decoy scan |
| `-Pn` | Skip ping — required when spoofing so nmap doesn't wait for ping reply |

---

### When to Use
- **Decoys** — when you want to make attribution harder during a real engagement
- **IP spoofing** — limited use, only when you control network monitoring
- **MAC spoofing** — internal network engagements where you're already on the same subnet

## Firewall and IDS Evasion

### Packet Fragmentation
Splits packets into smaller pieces to slip past firewalls and IDS that don't reassemble fragments before inspecting.

| Flag | Description |
|---|---|
| `-f` | Fragment packets into 8-byte chunks |
| `-ff` or `-f -f` | Fragment packets into 16-byte chunks |
| `--mtu [number]` | Custom fragment size — must be a multiple of 8 |
| `--data-length [num]` | Append random data to packets to make them look normal sized |

```bash
# Fragmented SYN scan on port 80
sudo nmap -sS -p80 -f TARGET_IP

# 16-byte fragments
sudo nmap -sS -p80 -ff TARGET_IP
```

---

### Quick Math for the Exam
- TCP header = 24 bytes
- `-f` = 8-byte fragments → 24 ÷ 8 = **3 fragments**
- `-ff` = 16-byte fragments → 24 ÷ 16 = **2 fragments** (16 bytes + 8 bytes)
- If TCP segment = 64 bytes and using `-ff` → 64 ÷ 16 = **4 fragments**

## Idle (Zombie) Scan

The stealthiest nmap scan — your IP never touches the target. All probes appear to come from a "zombie" host.

```bash
nmap -sI ZOMBIE_IP TARGET_IP

# Example using a printer as zombie
nmap -sI 10.10.5.5 TARGET_IP
```

**Answer to the question:** `-sI 10.10.5.5`

---

### How It Works
1. Attacker probes zombie host to record its current IP ID
2. Attacker sends SYN to target spoofed as coming from zombie IP
3. Attacker probes zombie again and checks if IP ID incremented

### Reading the Result
| IP ID Difference | Meaning |
|---|---|
| +1 | Port is closed or filtered — target sent RST to zombie, zombie ignored it |
| +2 | Port is open — target sent SYN/ACK to zombie, zombie responded with RST (incrementing IP ID) |

---

### Requirements
- Zombie must be idle — if it's busy generating its own traffic the IP ID increments are meaningless
- Good zombie candidates: rarely used printers, IoT devices, idle workstations
- You must be able to communicate with the zombie host

## Stealthy Scan Types
These work by setting TCP flags in unexpected ways to prompt responses from ports.

| Flag | Scan Type | Description |
|---|---|---|
| `-sN` | Null | No flags set — closed ports respond with RST |
| `-sF` | FIN | FIN flag only — closed ports respond with RST |
| `-sX` | Xmas | FIN+PSH+URG flags — closed ports respond with RST |
| `-sM` | Maimon | FIN+ACK — open and closed ports respond |
| `-sA` | ACK | ACK flag — maps firewall rules, doesn't detect open ports |
| `-sW` | Window | ACK with window size analysis — can detect open vs closed |
| `--scanflags URGACKPSHRSTSYNFIN` | Custom | Set any combination of TCP flags manually |

**Key distinction:**
- Null, FIN, Xmas → response from **closed** ports only (open ports stay silent)
- Maimon, ACK, Window → response from **both** open and closed ports

---

## Output and Verbosity

| Flag | Description |
|---|---|
| `--reason` | Shows why nmap classified a port as open/closed/filtered |
| `-v` | Verbose — shows results as they come in |
| `-vv` | Very verbose |
| `-d` | Debugging output |
| `-dd` | More detailed debugging |
| `--source-port PORT` | Spoof source port number (useful for bypassing some firewalls) |

**Tip:** Always use `--reason` when a scan gives unexpected results — it tells you exactly what response nmap got that led to its conclusion.

## Service and Version Detection

| Flag | Description |
|---|---|
| `-sV` | Detect service and version on open ports |
| `-sV --version-light` | Light probing — intensity 2, faster |
| `-sV --version-all` | Full probing — intensity 9, most complete |
| `--version-intensity [0-9]` | Manual intensity control |

**Important:** `-sV` forces a full TCP 3-way handshake — stealth SYN scan (`-sS`) is not possible when using `-sV`.

---

## OS Detection and Traceroute

| Flag           | Description                            |
| -------------- | -------------------------------------- |
| `-O`           | OS detection via TCP/IP fingerprinting |
| `--traceroute` | Map hops between you and the target    |

**TTL hints when OS detection is uncertain:**
- Linux = TTL 64
- Windows = TTL 128

**Note:** OS detection isn't always accurate — virtualisation, firewalls, and cloud networking can alter fingerprint results. Always support with additional recon.

---

## NSE — Nmap Scripting Engine

Scripts live at `/usr/share/nmap/scripts/` on Kali. Named by protocol (e.g. `http-*`, `ftp-*`, `smb-*`).

| Flag                        | Description                        |
| --------------------------- | ---------------------------------- |
| `-sC` or `--script=default` | Run default scripts                |
| `--script=[name]`           | Run a specific script              |
| `--script="ftp*"`           | Run all scripts matching a pattern |

### Script Categories

| Category    | Description                            |
| ----------- | -------------------------------------- |
| `auth`      | Authentication checks                  |
| `brute`     | Brute force password attacks           |
| `default`   | Safe, commonly useful scripts          |
| `discovery` | Enumerate info — DNS, databases, etc.  |
| `exploit`   | Attempt to exploit vulnerabilities     |
| `intrusive` | Aggressive — brute force, exploitation |
| `malware`   | Scan for backdoors                     |
| `safe`      | Won't crash the target                 |
| `version`   | Service version detection              |
| `vuln`      | Check for known vulnerabilities        |

**Caution:** `intrusive`, `exploit`, and `brute` scripts can crash services or trigger alerts. Only run on authorised targets.

---

## Output Formats

| Flag              | Format        | Use Case                                       |
| ----------------- | ------------- | ---------------------------------------------- |
| `-oN scan.nmap`   | Normal        | Human readable, default choice                 |
| `-oG scan.gnmap`  | Grepable      | Fast filtering with grep across multiple hosts |
| `-oX scan.xml`    | XML           | Processing output in other tools               |
| `-oA scan`        | All three     | Saves normal, grepable, and XML at once        |
| `-oS scan.kiddie` | Script kiddie | Useless — just for laughs                      |

**Tip:** Always use `-oA` on real engagements — you never know which format you'll need later.

---

## The -A Flag (All in One)
```bash
nmap -A TARGET_IP
# Equivalent to: -sV -O -sC --traceroute
```
Good for a thorough single-command scan. Noisy — don't use when stealth matters.

---

## Grep Trick for Grepable Output
```bash
# Find all hosts with HTTP open across multiple scan files
grep http scan.gnmap

# Returns full line including IP — much more useful than grepping normal output
```

## Host Discovery When ICMP is Filtered

When ICMP echo is blocked, follow this progression:

1. **ICMP timestamp** `-PP` — try ICMP alternatives before abandoning the protocol
2. **UDP ping** `-PU` — UDP to closed ports triggers ICMP port unreachable responses
3. **TCP ACK ping** `-PA` — pure TCP, no ICMP dependency, requires root
4. **Port enumeration** `-p [ports]` — verify discovered hosts with targeted scan