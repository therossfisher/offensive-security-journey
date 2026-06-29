# Linux Hardening

Quick reference for hardening a Linux system in a real environment. Covers the core checklist items, key commands, and config file locations. Not exhaustive — use this to get started and know what to look for.

---

## Mental Model — Defense in Depth

Hardening is layered. No single control is sufficient. Work through these layers in order:

1. Physical security
2. Disk encryption
3. Firewall
4. Remote access (SSH)
5. User accounts and privileges
6. Services and packages
7. Updates and patching
8. Logging and monitoring

---

## Physical Security

If an attacker has physical access, most software controls can be bypassed.

**The GRUB problem:** With physical access to the keyboard, an attacker can use GRUB (bootloader) to reset the root password. "Boot access = root access."

**Mitigations:**
- BIOS/UEFI boot password — prevents unauthorized boot, only practical for personal systems not servers
- GRUB password — prevents accessing advanced boot options

```bash
# Generate a GRUB password hash
grub2-mkpasswd-pbkdf2
# Add the resulting hash to the GRUB config file for your distro
```

PBKDF2 = Password-Based Key Derivation Function 2 — stretches the password with a salt and iteration count to make brute force harder.

> Cloud VMs: GRUB passwords don't apply — no physical terminal access.

---

## Disk Encryption — LUKS

LUKS = Linux Unified Key Setup. Standard for full disk encryption on Linux. Ships with most modern distros.

**Why it matters:** Stolen disk = useless without the key. Encrypted data should be as good as destroyed hardware.

**LUKS disk layout:**
- LUKS Partition Header (phdr) — stores UUID, cipher, cipher mode, key length, master key checksum
- Key Material slots (KM1–KM8) — master key encrypted with up to 8 different user passwords
- Bulk Data — actual encrypted data, encrypted with the master key

**Setting up LUKS from scratch:**
```bash
apt install cryptsetup                          # install if needed
fdisk -l                                        # identify partition
lsblk                                           # alternative partition view
cryptsetup -y -v luksFormat /dev/sdb1          # encrypt partition
cryptsetup luksOpen /dev/sdb1 EDCdrive         # create device mapping
ls -l /dev/mapper/EDCdrive                     # confirm mapping
dd if=/dev/zero of=/dev/mapper/EDCdrive        # overwrite with zeros
mkfs.ext4 /dev/mapper/EDCdrive -L "label"     # format
mount /dev/mapper/EDCdrive /media/secure-USB   # mount
```

**Working with an existing encrypted volume (image file or partition):**
```bash
sudo cryptsetup open --type luks /path/to/file.img myvault   # decrypt and map
mkdir ~/myvault                                               # create mount point
sudo mount /dev/mapper/myvault ~/myvault                     # mount
ls ~/myvault                                                  # read contents

# Cleanup — always do this when done
sudo umount ~/myvault
sudo cryptsetup close myvault
```

**Inspect LUKS settings on an encrypted volume:**
```bash
cryptsetup luksDump /dev/sdb1
# Shows: UUID, cipher (usually aes-xts-plain64), key size, PBKDF algorithm
```

---

## Firewall Configuration

Linux firewall stack: **netfilter** (kernel) → **iptables or nftables** (management layer) → **ufw or firewalld** (front-end).

For most practical hardening, use **ufw** unless you need fine-grained control.

### UFW — Uncomplicated Firewall (recommended for most cases)

```bash
ufw status                          # check current status and rules
ufw enable                          # turn on
ufw allow 22/tcp                    # allow SSH
ufw allow 80/tcp                    # allow HTTP
ufw allow 443/tcp                   # allow HTTPS
ufw deny 23/tcp                     # block Telnet
ufw delete allow 80/tcp             # remove a rule
ufw default deny incoming           # block all inbound by default
ufw default allow outgoing          # allow all outbound by default
```

### iptables — Lower level, more control

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # allow SSH inbound
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT   # allow SSH outbound
iptables -A INPUT -j DROP                         # block everything else inbound
iptables -A OUTPUT -j DROP                        # block everything else outbound
iptables -F                                       # flush all rules (start clean)
iptables -L                                       # list current rules
```

**Rule order matters in iptables** — first matching rule wins. Put specific allows before the catch-all DROP.

### nftables — Modern replacement for iptables

```bash
nft add table fwfilter
nft add chain fwfilter fwinput { type filter hook input priority 0 \; }
nft add chain fwfilter fwoutput { type filter hook output priority 0 \; }
nft add fwfilter fwinput tcp dport 22 accept
nft add fwfilter fwoutput tcp sport 22 accept
nft list table fwfilter                           # inspect rules
```

### Firewall Policy Approaches
- **Default deny (whitelist)** — block everything, allow specific exceptions. Tighter security, more maintenance.
- **Default allow (blacklist)** — allow everything, block specific exceptions. Easier to manage, higher risk.

Default deny is the correct choice for production security environments.

### Check what ports are open and what firewall rules exist:
```bash
ss -tlnp                            # listening TCP ports and what's running them
ss -ulnp                            # listening UDP ports
ufw status verbose                  # UFW rules
iptables -L -n -v                   # iptables rules with packet counts
```

---

## Remote Access — SSH Hardening

SSH config file: `/etc/ssh/sshd_config`

**Key hardening settings:**
```
PermitRootLogin no              # never allow direct root login over SSH
PasswordAuthentication no       # disable password auth, require keys
PubkeyAuthentication yes        # enable public key authentication
Port 2222                       # change default port (minor obscurity, reduces noise)
AllowUsers username             # whitelist specific users
MaxAuthTries 3                  # limit login attempts before disconnect
```

After editing sshd_config:
```bash
sudo systemctl restart sshd     # apply changes
```

> Always verify you have working key-based auth AND console/physical access before disabling password auth. Locking yourself out of a remote system is a bad day.

**Generate and deploy SSH keys:**
```bash
ssh-keygen -t rsa               # generate RSA key pair (id_rsa + id_rsa.pub)
ssh-keygen -t ed25519           # preferred — smaller, faster, more secure
ssh-copy-id username@server     # copy public key to remote system
```

**Find the flag / check SSH config:**
```bash
cat /etc/ssh/sshd_config
grep -i "permit\|auth\|password" /etc/ssh/sshd_config
```

---

## User Accounts and Privilege Management

### Check sudoers group membership
```bash
getent group sudo               # Debian/Ubuntu — shows sudo group members
getent group wheel              # RHEL/Fedora — wheel is the sudoers group
cat /etc/sudoers                # full sudoers config (requires root)
groups username                 # check groups for a specific user
```

### Add user to sudoers group
```bash
usermod -aG sudo username       # Debian/Ubuntu
usermod -aG wheel username      # RHEL/Fedora
```

### Disable accounts (without deleting them)
```bash
# Edit /etc/passwd and change the shell to /sbin/nologin
# Before: michael:x:1000:1000:Michael:/home/michael:/usr/bin/bash
# After:  michael:x:1000:1000:Michael:/home/michael:/sbin/nologin

# Do this for:
# - Root account (after creating a sudo user)
# - Inactive user accounts
# - Service accounts (www-data, nginx, mongo) — they need to run but never need a shell
```

### Enforce strong password policy

**Debian/Ubuntu** — `/etc/pam.d/common-password` (install with `apt install libpam-pwquality`)

**RHEL/Fedora** — `/etc/security/pwquality.conf`

```
difok=5         # characters in new password not in old password
minlen=10       # minimum password length
minclass=3      # minimum character classes (upper, lower, digits, special)
badwords=       # space-separated list of forbidden words
retry=3         # prompts before returning error
```

---

## Services and Packages

**Attack surface principle:** every installed package is potential vulnerability surface. Minimize what's installed.

```bash
# List running services
systemctl list-units --type=service --state=running

# Disable a service
systemctl disable servicename
systemctl stop servicename

# Check what's listening on the network
ss -tlnp
netstat -tlnp                   # older systems

# Remove a package entirely
apt remove packagename          # Debian/Ubuntu
dnf remove packagename          # Fedora/RHEL
```

**Legacy protocols to eliminate:**
- Telnet → replace with SSH
- FTP → replace with SFTP or FTPS
- TFTP → replace with SFTP
- HTTP (for admin) → replace with HTTPS

**Remove identification strings** from service banners where possible — version info leakage tells attackers what to target.

---

## Updates and Patching

Keep the OS, packages, AND kernel updated. Unpatched kernels are a real vector (see Dirty COW — CVE-2016-5195, local privilege escalation in all Linux kernels from 2.6.22 onward).

```bash
# Debian/Ubuntu
apt update && apt upgrade

# Fedora / modern RHEL (8+)
dnf update

# Older RHEL/CentOS (7 and earlier)
yum update
```

**Distribution lifecycle awareness:**
- Ubuntu LTS — 5 years free security updates, 10 years with Ubuntu Pro
- RHEL 8/9 — 12 years total (5 full + 5 maintenance + 2 extended)
- Running an EOL distro = no patches = open vulnerabilities

**Acronyms:**
- yum = Yellowdog Updater, Modified
- dnf = Dandified YUM

---

## Logging and Monitoring

Key log files — all in `/var/log/`:

| Log File | Contents | Distro |
|---|---|---|
| `/var/log/messages` | General system log | Most distros |
| `/var/log/auth.log` | All authentication attempts | Debian/Ubuntu |
| `/var/log/secure` | All authentication attempts | RHEL/Fedora |
| `/var/log/utmp` | Currently logged-in users | Most distros |
| `/var/log/wtmp` | Historical login/logout records | Most distros |
| `/var/log/kern.log` | Kernel messages | Most distros |
| `/var/log/boot.log` | Startup and boot messages | Most distros |

```bash
tail -n 20 /var/log/auth.log            # last 20 lines of auth log
tail -f /var/log/auth.log               # live follow — watch in real time
grep "Failed password" /var/log/auth.log  # find failed login attempts
grep "FAILED" /var/log/secure           # RHEL equivalent
grep denied /var/log/kern.log           # kernel denials

# Check sources.list (package sources config)
cat /etc/apt/sources.list
```

---

## Hardening Checklist — Quick Reference

Use this as a starting point before going deeper:

- [ ] Physical access secured or GRUB password set
- [ ] Full disk encryption enabled (LUKS) on sensitive systems
- [ ] Firewall enabled, default deny inbound
- [ ] Only required ports open — verify with `ss -tlnp`
- [ ] SSH: root login disabled, password auth disabled, key auth enabled
- [ ] No unnecessary users in sudo/wheel group
- [ ] Root account disabled or restricted
- [ ] Password complexity policy enforced
- [ ] Telnet, FTP, TFTP disabled or removed
- [ ] Unnecessary services stopped and disabled
- [ ] OS and kernel fully patched and on a supported release
- [ ] Log files accessible and being reviewed (or fed to SIEM)
- [ ] Service accounts set to /sbin/nologin

---

## Interview Talking Points

**"How would you harden an SSH server?"**
Disable root login, disable password auth, enable key-based auth only, change default port, restrict AllowUsers, set MaxAuthTries. Show you know the specific sshd_config directives.

**"What's your approach to firewall configuration on Linux?"**
Default deny inbound. Allow only what's required. Use ufw for simplicity, iptables/nftables when you need granular control. Verify with ss -tlnp after changes.

**"How do you handle privilege management on Linux?"**
Never run services as root. Disable root login. Use sudo for administrative tasks with a dedicated admin account. Regularly audit sudo group membership. Disable accounts that no longer need access.

**"What logs would you check if you suspected unauthorized access?"**
/var/log/auth.log (Debian) or /var/log/secure (RHEL), grep for Failed password, check /var/log/wtmp for login history, check currently logged-in users with who or w.
