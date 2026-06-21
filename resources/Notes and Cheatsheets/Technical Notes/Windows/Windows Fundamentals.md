# Windows Fundamentals — Study Notes

---

## OS Basics

- Current desktop OS: **Windows 11** (Home and Pro editions)
- Current server OS: **Windows Server 2025**
- Pro-exclusive feature: **BitLocker** drive encryption
- File system: **NTFS** (New Technology File System)
  - Supports files >4GB, per-file/folder permissions, compression, encryption (EFS), journaling

### NTFS Permissions
Full Control / Modify / Read & Execute / List Folder Contents / Read / Write

Right-click file/folder → Properties → **Security** tab to view permissions

### Alternate Data Streams (ADS)
Every NTFS file has at least one data stream (`$DATA`). ADS lets files contain hidden additional streams.
Used legitimately (e.g. "downloaded from internet" flag) and maliciously (malware hiding data).
View with PowerShell — covered in PowerShell notes.

---

## Key System Locations

| Path | Purpose |
|------|---------|
| `C:\Windows` | Windows OS files |
| `C:\Windows\System32` | Critical OS files — **do not touch** |
| `C:\Users\<username>` | User profile folders |
| `%windir%` | Environment variable for Windows directory |

---

## User Accounts & Groups

Two types: **Administrator** and **Standard User**

- Admins: can modify system, add/remove users, install software
- Standard: changes limited to their own files/folders

**Access via:** Start Menu → type "Other User" → System Settings  
**Or:** Run dialog → `lusrmgr.msc` (Local Users and Groups)

### User Account Control (UAC)
Introduced in Vista. Prompts for elevation when system-level changes are needed even for admins.
Four levels (slider in UAC settings):
- Always notify
- Notify for apps only *(default)*
- Notify without dimming screen
- Never notify *(not recommended)*

Built-in local Administrator account is exempt from UAC by default.

---

## Important Utilities & Run Commands

| Utility | Command |
|---------|---------|
| System Configuration | `msconfig` |
| Computer Management | `compmgmt.msc` |
| Local Users & Groups | `lusrmgr.msc` |
| System Information | `msinfo32.exe` |
| Resource Monitor | `resmon.exe` |
| Registry Editor | `regedit` / `regedt32.exe` |
| UAC Settings | `UserAccountControlSettings.exe` |
| Control Panel | `control.exe` |
| Task Manager | `Ctrl+Shift+Esc` |
| Event Viewer | Inside `compmgmt.msc` |
| Disk Management | Inside `compmgmt.msc` |
| Windows Update | `control /name Microsoft.WindowsUpdate` |

**Open Run dialog:** `Win + R`

---

## System Configuration (msconfig)

Five tabs: General / Boot / Services / Startup / Tools  
Used for **troubleshooting startup issues** — not a startup manager.  
Startup items managed via **Task Manager** on Windows 10/11.  
On Windows Server: startup items live in the **Startup folder** (`shell:startup` in Run).

---

## Computer Management (compmgmt.msc)

Three sections:

**System Tools**
- Task Scheduler — create/manage automated tasks
- Event Viewer — audit log for system events (5 event types: Critical, Error, Warning, Information, Verbose)
- Shared Folders — view shares, sessions, open files
- Local Users and Groups — same as lusrmgr.msc
- Performance Monitor (perfmon) — real-time/logged performance data
- Device Manager — view/configure hardware

**Storage**
- Disk Management — partition management, drive letters, extend/shrink volumes

**Services and Applications**
- Services — view all services, start/stop, set startup type (Automatic/Manual/Disabled)
- WMI Control — manages Windows Management Instrumentation

---

## Windows Security Areas

- **Virus & Threat Protection** — Defender AV, real-time protection, quarantine, ransomware protection
- **Firewall & Network Protection** — domain/private/public profiles
- **App & Browser Control** — SmartScreen settings (Block/Warn/Off)
- **Device Security** — Core isolation, Memory Integrity
- **TPM** — Trusted Platform Module: hardware-based crypto processor for security functions

**Status icons:** Green = good / Yellow = recommendation / Red = immediate attention

### BitLocker
Drive encryption tied to TPM. On systems without TPM 1.2+, requires a **startup key** on a removable drive.

---

## Volume Shadow Copy Service (VSS)

Point-in-time snapshots of data. Stored in `System Volume Information` folder.  
Enables: restore points, system restore, restore settings, delete restore points.  
Ransomware commonly deletes shadow copies to prevent recovery.

---

## CMD — Key Commands

| Command | Purpose |
|---------|---------|
| `hostname` | Shows computer name |
| `whoami` | Shows current user |
| `ver` | OS version |
| `systeminfo` | Full system info |
| `ipconfig` | Network config |
| `ipconfig /all` | Detailed network config (MAC, DHCP, DNS) |
| `ipconfig /?` | Help for ipconfig |
| `ping <target>` | Test connectivity |
| `tracert <target>` | Trace network route |
| `nslookup <domain>` | DNS lookup |
| `netstat` | Active connections |
| `netstat -abon` | All connections + owning process + PID |
| `tasklist` | Running processes |
| `tasklist /FI "imagename eq notepad.exe"` | Filter processes by name |
| `taskkill /PID <pid>` | Kill process by PID |
| `net help` | Help for net subcommands |
| `cls` | Clear screen |
| `shutdown /s` | Shut down |
| `shutdown /r` | Restart |
| `shutdown /a` | Abort scheduled shutdown |
| `<command> /?` | Help for most commands |
| `<command> \| more` | Page through long output |

### File/Directory Commands (CMD)

| Command | Purpose |
|---------|---------|
| `cd` | Show current directory |
| `cd <dir>` | Change directory |
| `cd ..` | Go up one level |
| `dir` | List directory contents |
| `dir /a` | Include hidden/system files |
| `dir /s` | Recurse subdirectories |
| `tree` | Visual directory tree |
| `mkdir <name>` | Create directory |
| `rmdir <name>` | Remove directory |
| `copy <src> <dest>` | Copy file |
| `move <src> <dest>` | Move/rename file |
| `del <file>` / `erase <file>` | Delete file |
| `type <file>` | Display file contents |
| `more <file>` | Page through file |

Wildcard `*` works across commands: `copy *.txt C:\Backup`

---

## Active Directory (AD) Basics

**Windows Domain** — centralized group of users/computers managed via **Active Directory Domain Services (AD DS)**  
**Domain Controller (DC)** — server running AD DS; stores all credentials

### AD Object Types
- **Users** — people or service accounts; security principals
- **Machines** — joined computers; account name = `ComputerName$` (e.g. `TOM-PC$`)
- **Security Groups** — grant permissions to resources; users can be in multiple groups

### Key Default Groups

| Group | Role |
|-------|------|
| Domain Admins | Full admin over entire domain |
| Server Operators | Administer DCs (no group changes) |
| Backup Operators | Access any file for backup |
| Account Operators | Create/modify user accounts |
| Domain Users | All domain users |
| Domain Computers | All domain computers |
| Domain Controllers | All DCs |

### OUs vs Security Groups
- **OUs** — apply policies (GPOs); user can only be in **one** OU
- **Security Groups** — grant permissions to resources; user can be in **many** groups

### Organizational Units (OUs)
Managed via: Start Menu → **Active Directory Users and Computers**  
Default containers: Builtin / Computers / Domain Controllers / Users / Managed Service Accounts

To delete a protected OU: View → **Advanced Features** → Properties → Object tab → uncheck deletion protection

**Delegation** — grant specific users control over an OU (e.g. IT support resetting passwords) without full Domain Admin rights

---

## Group Policy Objects (GPOs)

Collections of settings applied to OUs. Can target users or computers.  
Managed via: **Group Policy Management** tool  
GPOs apply to linked OU **and all child OUs**.  
Distributed via network share: **SYSVOL** (`C:\Windows\SYSVOL\sysvol\`)

Force immediate sync:
```
gpupdate /force
```

Common uses: password policies, screen lock timeouts, restricting Control Panel access

---

## Authentication Protocols

### Kerberos *(default, modern Windows)*
Ticket-based system:
1. User authenticates to KDC → receives **TGT** (Ticket Granting Ticket)
2. TGT used to request **TGS** (Ticket Granting Service) for specific services
3. TGS presented to service to establish connection

Password never transmitted over network.

### NetNTLM *(legacy, kept for compatibility)*
Challenge-response mechanism — server sends random challenge, client responds using password hash.  
Password hash never transmitted directly.

---

## AD Structure — Scaling Up

| Concept | Description |
|---------|-------------|
| **Tree** | Multiple domains sharing the same namespace (e.g. `uk.thm.local`, `us.thm.local`) |
| **Forest** | Multiple trees with different namespaces joined together |
| **Trust Relationship** | Allows users in one domain to access resources in another |
| **Enterprise Admins** | Admin privileges across all domains in a forest |

Trust can be **one-way** or **two-way**. Joining domains in a tree/forest creates two-way trust by default.  
Trust ≠ automatic access — access still has to be explicitly authorized.

---

## Notes / Things to Add

*Use this space as you pick up new stuff:*

-
-
-
