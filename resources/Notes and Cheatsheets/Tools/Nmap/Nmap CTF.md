## CTF / HTB / THM Speed Scanning

In real engagements you scan slow and quiet — IDS evasion, low noise, blending in. In CTF environments like HTB and THM, the boxes are isolated lab machines with no IDS watching. Nobody cares if you're loud. The goal is to find open ports fast and move on to actual exploitation.

These flags are **not appropriate for real pentests or production networks.** They're optimized purely for speed in controlled lab environments.

---

### The Two-Phase CTF Workflow

The fastest reliable approach is two scans in sequence — a fast full port scan first to find everything, then a targeted service scan on only the open ports.

**Phase 1 — Fast full port scan (find everything)**
```bash
sudo nmap -p- --min-rate 5000 -Pn TARGET_IP
```

- `-p-` — scans all 65,535 ports instead of just the top 1000. Essential because CTF boxes frequently put services on non-standard ports (8080, 8443, 3000, etc.)
- `--min-rate 5000` — sends at least 5000 packets per second. Dramatically faster than default. On HTB/THM this is fine — on a real network this is a flood
- `-Pn` — skips host discovery ping. HTB/THM boxes sometimes don't respond to pings even when they're up, which causes nmap to skip them entirely. This tells nmap to assume the host is alive and scan it regardless

**Phase 2 — Targeted service scan on open ports only**
```bash
sudo nmap -sC -sV -Pn -p 22,80,3000 TARGET_IP -oA nmap/targeted
```

- Take whatever ports Phase 1 found and put them in `-p`
- `-sC` — runs default scripts (grabs banners, checks for common vulns, identifies services)
- `-sV` — version detection
- `-oA nmap/targeted` — saves output in all three formats for your notes

**Why two phases instead of one?**
Running `-sC -sV` against all 65,535 ports is slow because version detection and scripts take time per port. Running a raw fast scan first to find which ports are open, then running the slower detailed scan on only those ports, gets you full coverage much faster.

---

### Quick Reference

```bash
# Phase 1 — fast full port discovery
sudo nmap -p- --min-rate 5000 -Pn TARGET_IP

# Phase 2 — service/version/script on open ports
sudo nmap -sC -sV -Pn -p PORT,PORT,PORT TARGET_IP -oA nmap/targeted

# One-liner if you want both at once (slower but one command)
sudo nmap -p- -sC -sV --min-rate 5000 -Pn TARGET_IP -oA nmap/full
```

---

### Key Flags Explained

| Flag | What it does | Why it matters in CTF |
|---|---|---|
| `-p-` | Scan all 65,535 ports | CTF boxes hide services on non-standard ports constantly |
| `--min-rate 5000` | Floor of 5000 packets/sec | Makes full port scan take seconds instead of minutes |
| `-Pn` | Skip ping, assume host is up | HTB/THM boxes often block ICMP — without this nmap skips them |
| `--open` | Only show open ports in output | Cuts noise — filtered/closed ports are irrelevant in CTF |
| `-T4` | Aggressive timing preset | Alternative to `--min-rate`, slightly less aggressive |

---

### Flags to Avoid in CTF (but use in real engagements)

| Flag | Why you'd skip it in CTF |
|---|---|
| `-T1` / `-T2` | Paranoid/sneaky timing — unnecessary when there's no IDS |
| `--scan-delay` | Deliberate slowdown for evasion — not needed |
| `-D` (decoy) | Spoofed source IPs — pointless on an isolated lab network |
| `-f` (fragment packets) | Firewall/IDS evasion — no firewall to evade in lab |

---

### Saving Output

Always save nmap output — you'll reference it throughout the box.

```bash
# Create the directory first
mkdir -p nmap

# All three formats at once (.nmap, .xml, .gnmap)
sudo nmap -p- --min-rate 5000 -Pn TARGET_IP -oA nmap/initial

# View saved output
cat nmap/initial.nmap
```

The `.xml` output is useful if you later pipe results into other tools. The `.nmap` is the human-readable version you'll actually read.