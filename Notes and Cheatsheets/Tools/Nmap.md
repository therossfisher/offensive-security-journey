# Nmap Cheat Sheet

## Syntax
```
nmap [options] [target]
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

## Notes
- Always save output with `-oN` — you'll want to reference it later
- Run full port scan (`-p-`) when initial scan comes back with nothing interesting
- Use inside msfconsole with `db_nmap` to auto-save results to database
