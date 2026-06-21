# PowerShell — Study Notes

---

## What Is PowerShell?

Cross-platform task automation tool built on the **.NET framework**.  
Unlike CMD (text-based), PowerShell is **object-oriented** — commands pass objects with properties and methods, not just raw text.  
Available on Windows, macOS, and Linux (PowerShell Core, released 2016).

Launch from CMD: type `powershell` and hit Enter.  
Prompt looks like: `PS C:\Users\you>`

---

## Cmdlet Basics

Commands in PowerShell are called **cmdlets** (pronounced "command-lets").  
Follow a consistent **Verb-Noun** naming convention.

```
Get-Content      # Reads a file
Set-Location     # Changes directory
Remove-Item      # Deletes a file or folder
```

---

## Essential Cmdlets

### Discovery & Help

| Cmdlet | Purpose |
|--------|---------|
| `Get-Command` | List all available cmdlets, functions, aliases |
| `Get-Command -Name Remove*` | Filter by name pattern |
| `Get-Command -CommandType Function` | Filter by type |
| `Get-Help <cmdlet>` | Show help for a cmdlet |
| `Get-Help <cmdlet> -examples` | Show usage examples |
| `Get-Help <cmdlet> -detailed` | Full detail |
| `Get-Alias` | List all aliases (e.g. `dir` → `Get-ChildItem`) |
| `Find-Module -Name "pattern*"` | Search PowerShell Gallery for modules |
| `Install-Module -Name "<name>"` | Install a module from online repository |

---

## File System Navigation

| Cmdlet | Equivalent | Purpose |
|--------|------------|---------|
| `Get-ChildItem` | `dir` / `ls` | List directory contents |
| `Get-ChildItem -Path C:\Users` | | List specific path |
| `Set-Location -Path .\Documents` | `cd` | Change directory |
| `Get-Content -Path .\file.txt` | `type` / `cat` | Read file contents |
| `New-Item -Path .\file.txt -ItemType File` | `touch` | Create file |
| `New-Item -Path .\folder -ItemType Directory` | `mkdir` | Create directory |
| `Remove-Item -Path .\file.txt` | `del` | Delete file |
| `Remove-Item -Path .\folder` | `rmdir` | Delete directory |
| `Copy-Item -Path .\src -Destination .\dest` | `copy` | Copy file/folder |
| `Move-Item -Path .\src -Destination .\dest` | `move` | Move/rename file/folder |

---

## Piping

PowerShell passes **objects** through pipes, not just text — much more powerful than CMD piping.

```powershell
Get-ChildItem | Sort-Object Length
```

---

## Filtering & Sorting

| Cmdlet | Purpose |
|--------|---------|
| `Sort-Object <Property>` | Sort output by a property |
| `Where-Object -Property <prop> -eq "value"` | Filter by property value |
| `Select-Object Name, Length` | Show only specific properties |
| `Select-String -Path .\file.txt -Pattern "word"` | Search file for text pattern (like grep) |

### Comparison Operators

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-gt` | Greater than (strict) |
| `-ge` | Greater than or equal to |
| `-lt` | Less than (strict) |
| `-le` | Less than or equal to |
| `-like "pattern*"` | Wildcard match |

**Example — filter .txt files:**
```powershell
Get-ChildItem | Where-Object -Property Extension -eq ".txt"
```

**Example — files larger than 100 bytes:**
```powershell
Get-ChildItem | Where-Object -Property Length -gt 100
```

**Example — find file by name pattern:**
```powershell
Get-ChildItem | Where-Object -Property Name -like "ship*"
```

**Example — chain multiple cmdlets:**
```powershell
Get-ChildItem | Sort-Object Length -Descending | Select-Object -First 1
```

---

## System & Network Information

| Cmdlet | Purpose |
|--------|---------|
| `Get-ComputerInfo` | Comprehensive system info (OS, hardware, BIOS) |
| `Get-LocalUser` | List local user accounts + status |
| `Get-NetIPConfiguration` | Network interfaces, IPs, DNS, gateway |
| `Get-NetIPAddress` | All IP addresses on all interfaces |
| `Get-Process` | Running processes with CPU/memory usage |
| `Get-Service` | All services and their status (Running/Stopped) |
| `Get-NetTCPConnection` | Active TCP connections and listening ports |
| `Get-FileHash -Path .\file.txt` | Generate SHA256 hash of a file |

**Useful for blue team / forensics:**
- `Get-Service` → spot anomalous/tampered services
- `Get-NetTCPConnection` → find hidden backdoors or unexpected connections
- `Get-Process` → identify suspicious running processes
- `Get-FileHash` → verify file integrity

---

## Alternate Data Streams (ADS)

View ADS attached to a file:
```powershell
Get-Item -Path "C:\path\file.txt" -Stream *
```

Output will show `:$DATA` (normal content) and any hidden streams by name.

---

## Active Directory via PowerShell

Reset a user's password (requires AD delegation):
```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```

Force password change at next logon:
```powershell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

Force GPO update on local machine:
```powershell
gpupdate /force
```

---

## Remote Execution

```powershell
# Run a script on a remote machine
Invoke-Command -FilePath C:\scripts\script.ps1 -ComputerName Server01

# Run a single command on a remote machine
Invoke-Command -ComputerName Server01 -Credential Domain01\User01 -ScriptBlock { Get-Culture }

# No credentials needed example
Invoke-Command -ComputerName RoyalFortune -ScriptBlock { Get-Service }
```

`Invoke-Command` is heavily used by both sysadmins (remote management) and red teamers (executing payloads remotely).

---

## Key Aliases (CMD → PowerShell)

| Alias | PowerShell Cmdlet |
|-------|------------------|
| `dir` / `ls` | `Get-ChildItem` |
| `cd` | `Set-Location` |
| `cat` / `type` | `Get-Content` |
| `echo` | `Write-Output` |
| `cp` | `Copy-Item` |
| `mv` | `Move-Item` |
| `rm` / `del` | `Remove-Item` |
| `mkdir` | `New-Item -ItemType Directory` |
| `cls` / `clear` | `Clear-Host` |

---

## Scripting Notes

PowerShell scripts = `.ps1` files containing sequences of cmdlets/commands.

**Blue team uses:** log analysis, IOC extraction, anomaly detection, malware reverse engineering  
**Red team uses:** system enumeration, remote command execution, obfuscated bypass scripts  
**Sysadmin uses:** integrity checks, policy enforcement, automated incident response

---

## Notes / Things to Add

*Use this space as you pick up new stuff:*

-
-
-
