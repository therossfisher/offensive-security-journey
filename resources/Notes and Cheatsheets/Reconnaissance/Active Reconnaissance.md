# Active Reconnaissance

Direct engagement with target systems. Sends packets, makes connections, probes services. Leaves traces in logs, IDS alerts, WAF blocks, and honeypot triggers.

**Critical rule:** Never perform active recon without explicit, signed legal authorization (pentest contract or bug bounty scope). Unauthorized probing is illegal in most jurisdictions.

---

## Passive vs Active — Quick Distinction

| Type | Interaction | Detection Risk | Examples |
|---|---|---|---|
| Passive | None — public data only | None | DNS, WHOIS, Shodan, CT logs |
| Active | Direct contact with target | High | Ping, port scanning, banner grabbing, fuzzing |

Note: Direct interaction with a person affiliated with the target counts as active recon even without sending packets.

---

## Web Browser as a Recon Tool

The most convenient and least suspicious active recon tool. Traffic blends in with normal user activity.

### Default Ports
- HTTP → TCP port 80
- HTTPS → TCP port 443
- HTTP/3 → UDP port 443 (QUIC protocol — shows as `h3` in browser Network tab)
- Non-standard ports → specify explicitly: `https://target.com:8443/`

### Developer Tools — `Ctrl+Shift+I` (Windows/Linux) or `Option+Command+I` (macOS)

| Tab | What to Look For |
|---|---|
| **Network** | Request/response headers — `Server`, `X-Powered-By`, `Content-Security-Policy`, cookies |
| **Console** | Execute JavaScript, view errors, interact with DOM |
| **Sources** | JavaScript files — hardcoded API endpoints, internal service references, developer comments |
| **Application** | Cookies, Local Storage, Session Storage — session tokens, API keys, auth data |
| **Security** | Certificate details, SANs (Subject Alternative Names) — reveals additional subdomains |

### Useful Browser Extensions
| Extension | Purpose |
|---|---|
| FoxyProxy | Switch between proxies — Burp Suite, ZAP, SOCKS5 |
| User-Agent Switcher | Emulate different browsers/OS to find mobile endpoints |
| Wappalyzer | Passive tech stack fingerprinting — CMS, web server, JS frameworks, CDN |
| BuiltWith | Alternative to Wappalyzer, sometimes detects different tech |

---

## Ping

Tests host reachability using ICMP Echo Request (type 8) / Echo Reply (type 0).

```bash
# Linux/macOS — send 5 packets
ping -c 5 TARGET_IP

# Windows
ping -n 5 TARGET_IP

# Force IPv4 or IPv6
ping -4 -c 5 TARGET_IP
ping -6 -c 5 TARGET_IPV6
```

### Reading TTL for OS Fingerprinting
| TTL Value | Likely OS |
|---|---|
| 64 | Linux |
| 128 | Windows |

**Important:** TTL decrements by 1 at each hop. A TTL of 58 likely means Linux 6 hops away, not a different OS.

### Interpreting Results
| Result | Meaning | Next Step |
|---|---|---|
| Fast replies, low packet loss | Host is up, ICMP allowed | Proceed to port scanning |
| "Destination Host Unreachable" | Host down or no route | Check if machine is powered on |
| 100% packet loss, no error | ICMP filtered | Try TCP/UDP host discovery with Nmap |
| High latency / heavy loss | Congestion or filtering | Investigate with traceroute |

**Common reasons for no reply:** Host is off, firewall blocks ICMP, Windows Firewall (blocks ping by default), cloud providers (AWS/Azure/GCP often block ICMP), CDNs.

---

## Traceroute

Maps the network path from your system to a target by exploiting the TTL field. Each router decrements TTL by 1 — when it hits 0, the router drops the packet and sends back an ICMP Time-to-Live Exceeded message revealing its IP.

```bash
# Linux/macOS
traceroute TARGET_IP

# Windows
tracert TARGET_IP

# IPv6
traceroute -6 TARGET_IPV6
traceroute6 TARGET_IPV6

# TCP mode (bypass UDP filters)
traceroute -T TARGET_IP

# ICMP mode
traceroute -I TARGET_IP

# Real-time continuous view with stats
mtr TARGET_IP
```

### Reading Output
- Each numbered line = one hop (one router)
- Up to 3 IPs per hop = 3 probe packets, may take different paths (load balancing)
- `*` = router didn't respond (rate limiting, firewall, or ICMP suppression)
- Final hop IP matching destination = traceroute completed successfully

### Key Notes
- Routes are NOT fixed — dynamic routing, load balancing, and anycast mean consecutive runs can take completely different paths
- Many routers suppress ICMP responses intentionally — `*` doesn't mean the route is broken
- Use `mtr` for real-time path monitoring with per-hop packet loss and latency stats

---

## Telnet — Banner Grabbing (Legacy)

Telnet sends all data in cleartext including credentials. **Never use for actual remote administration.** Useful for understanding banner grabbing fundamentals and legacy environments.

```bash
# Connect to any TCP port
telnet TARGET_IP PORT

# Example — grab HTTP banner
telnet TARGET_IP 80
# Then type:
GET / HTTP/1.1
host: telnet
# Press Enter twice
```

**What you're looking for:** `Server:` header revealing web server type and version (e.g. `Server: nginx/1.6.2`)

**Limitation:** Cannot handle encrypted connections (HTTPS, SMTPS, etc.)

---

## Netcat — Banner Grabbing and Port Probing

More versatile than telnet. Supports TCP and UDP, can act as client or server.

```bash
# Banner grabbing (same technique as telnet)
nc TARGET_IP PORT

# HTTP banner grab
nc TARGET_IP 80
# Then type:
GET / HTTP/1.1
host: netcat
# Press Shift+Enter

# Listen on a port (server mode)
nc -lvnp 1234

# Connect to listener (client mode)
nc TARGET_IP 1234

# IPv6
nc -6 TARGET_IPV6 PORT
```

### Netcat Flags
| Flag | Meaning |
|---|---|
| `-l` | Listen mode |
| `-p` | Specify port number |
| `-n` | Numeric only — no DNS resolution |
| `-v` | Verbose |
| `-vv` | Very verbose |
| `-k` | Keep listening after client disconnects |

### FTP Banner Grab Example
```bash
nc TARGET_IP 21
# Server sends banner immediately — no commands needed
```

---

## curl — HTTP Banner Grabbing (Preferred over telnet)

```bash
# Grab HTTP headers only
curl -I http://TARGET_IP
curl -I https://TARGET_IP

# Verbose — shows full request and response
curl -v https://TARGET_IP

# For TLS-wrapped services
openssl s_client -connect TARGET_IP:443
```

---

## Banner Grabbing — What You're Looking For

Connect to a port → read what the server sends back → cross-reference with CVE/Exploit-DB

| Port | Protocol | What Banner Reveals |
|---|---|---|
| 21 | FTP | FTP server software and version |
| 22 | SSH | OpenSSH version |
| 25 | SMTP | Mail server software |
| 80 | HTTP | Web server type and version |
| 443 | HTTPS | Use curl or openssl |

---

## Quick Reference

| Command | Example |
|---|---|
| Ping (Linux) | `ping -c 10 TARGET_IP` |
| Ping (Windows) | `ping -n 10 TARGET_IP` |
| Traceroute (Linux) | `traceroute TARGET_IP` |
| Tracert (Windows) | `tracert TARGET_IP` |
| MTR real-time | `mtr TARGET_IP` |
| Traceroute TCP mode | `traceroute -T TARGET_IP` |
| Telnet banner grab | `telnet TARGET_IP PORT` |
| Netcat banner grab | `nc TARGET_IP PORT` |
| Netcat listener | `nc -lvnp PORT` |
| curl HTTP headers | `curl -I http://TARGET_IP` |
| TLS inspection | `openssl s_client -connect TARGET_IP:443` |

---

## Defender Notes
- Active probes surface in access logs, firewall logs, WAF events, IDS alerts
- To blend in: use realistic User-Agent strings, slow timing, normal browser behavior
- Rapid page loads, modified headers, and abnormal User-Agent strings are detection signals
- Modern WAFs, CDNs, SIEMs, and EDR solutions detect recon patterns quickly
