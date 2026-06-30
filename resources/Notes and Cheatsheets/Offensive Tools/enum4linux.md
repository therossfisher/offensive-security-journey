# enum4linux Cheatsheet
**Tool for SMB/Samba enumeration via null sessions**

---

## What It Does

enum4linux queries SMB services (ports 139/445) without credentials using null session authentication. Old SMB configurations allow anonymous connections that leak a surprising amount of information — usernames, shares, password policies, group memberships, and OS info.

Run it any time you see 139 or 445 open on a target.

---

## Basic Commands

```bash
# Full enumeration — run this first, always
enum4linux -a TARGET_IP

# Just usernames
enum4linux -U TARGET_IP

# Just shares
enum4linux -S TARGET_IP

# Just password policy
enum4linux -P TARGET_IP

# Just OS info
enum4linux -o TARGET_IP

# Just groups
enum4linux -G TARGET_IP

# RID cycling (enumerate users by brute forcing SID/RID combos)
enum4linux -r TARGET_IP

# With credentials if you have them
enum4linux -u USERNAME -p PASSWORD -a TARGET_IP

# Save output to file
enum4linux -a TARGET_IP | tee enum4linux_output.txt
```

---

## What to Look For in Output

### Users Section
```
S-1-22-1-1000 Unix User\kay (Local User)
S-1-22-1-1001 Unix User\jan (Local User)
```
These are your targets for brute forcing. Note the difference between Windows SIDs (S-1-5-21-...) and Unix UIDs (S-1-22-1-...). On Linux/Samba boxes the Unix users are what matter.

### Shares Section
```
Sharename       Type      Comment
Anonymous       Disk
IPC$            IPC       IPC Service
```
Always try to connect to any non-IPC$ share:
```bash
smbclient //TARGET_IP/Anonymous -N
```
`-N` means no password. List files with `ls`, download with `get filename`.

### Password Policy Section
```
Minimum password length: 5
Password Complexity: Disabled
Account Lockout Threshold: None
```
No lockout threshold = brute force freely without risk of locking accounts. Short minimum length and no complexity = rockyou will probably crack it. This is intel for Hydra.

### OS Info Section
```
BASIC2  Wk Sv PrQ Unx NT SNT Samba Server 4.15.13-Ubuntu
os version: 6.1
```
Tells you Samba version which can have its own vulnerabilities.

---

## Full Workflow for SMB Enumeration

```bash
# 1. Confirm SMB is open
nmap -sV -p 139,445 TARGET_IP

# 2. Run full enum4linux
enum4linux -a TARGET_IP | tee enum4linux_output.txt

# 3. Extract usernames (grep for cleaner output)
enum4linux -a TARGET_IP | grep "Unix User"

# 4. Check anonymous share access
smbclient -L //TARGET_IP -N
smbclient //TARGET_IP/SHARENAME -N

# 5. If you have creds, try authenticated enumeration
enum4linux -u USERNAME -p PASSWORD -a TARGET_IP
smbclient //TARGET_IP/SHARENAME -U USERNAME
```

---

## Common Findings and What They Mean

| Finding | What to Do |
|---------|-----------|
| Usernames via RID cycling | Use for Hydra SSH/SMB brute force |
| Anonymous share readable | Connect with smbclient, look for credentials/configs |
| No account lockout | Brute force without rate limiting |
| Short min password length | rockyou likely has the password |
| Samba version exposed | Check searchsploit for known vulns |
| Password complexity disabled | Simple passwords, try common ones first |

---

## Troubleshooting

**"Can't get OS info with smbclient"** — Normal, usually gets it another way

**"smbXcli_negprot_smb1_done: No compatible protocol"** — Server disabled SMB1, enum4linux still works via SMB2

**No users returned** — Try RID cycling specifically: `enum4linux -r TARGET_IP`

**Null session blocked** — Try with guest account: `enum4linux -u guest -p "" -a TARGET_IP`

---

## Related Tools

```bash
# Modern alternative to enum4linux (Python, more features)
enum4linux-ng -A TARGET_IP

# Direct SMB client
smbclient -L //TARGET_IP -N

# Nmap SMB scripts
nmap --script smb-enum-users,smb-enum-shares -p 139,445 TARGET_IP

# CrackMapExec (more powerful, good for AD environments)
crackmapexec smb TARGET_IP --users
crackmapexec smb TARGET_IP --shares
```

---

## Notes for Real Engagements

- Always save output: `enum4linux -a TARGET | tee output.txt`
- Document every username found — cross-reference against other services (FTP, web app login, SSH)
- Check if Anonymous share has write access — if so, you can potentially plant files
- Password policy intel directly informs your brute force strategy
- RID cycling (S-1-22-1 SIDs) is specifically for Unix/Linux users on Samba — don't confuse with Windows domain SIDs
