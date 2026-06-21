# Linux Fundamentals — Study Notes

---

## Core Concepts

Linux is an umbrella term for multiple OSs based on UNIX. Open source, highly customizable.
Common distros: **Ubuntu**, Debian, Kali. Ubuntu can run on as little as 512MB RAM.

Powers: web servers, smart cars, Android, supercomputers, PoS systems, traffic systems, and more.

---

## Essential Commands

### Navigation & Filesystem

| Command | What it does |
|--------|--------------|
| `ls` | List contents of current directory |
| `ls -a` | Include hidden files (prefixed with `.`) |
| `ls -lh` | Long format, human-readable sizes + permissions |
| `cd <dir>` | Change directory |
| `pwd` | Print current working directory (full path) |
| `cat <file>` | Output file contents |
| `find -name <file>` | Search for file by name |
| `find -name *.txt` | Search by extension (wildcard) |
| `grep "value" <file>` | Search file contents for a value |
| `grep -R "value" <dir>` | Recursive grep across all files in a directory |
| `wc -l <file>` | Count lines in a file |

### File Management

| Command | What it does |
|--------|--------------|
| `touch <file>` | Create empty file |
| `mkdir <dir>` | Create directory |
| `cp <src> <dest>` | Copy file |
| `mv <src> <dest>` | Move or rename file |
| `rm <file>` | Delete file |
| `rm -R <dir>` | Delete directory recursively |
| `file <file>` | Determine file type |

### Output & Text Editing

| Command | What it does |
|--------|--------------|
| `echo "text"` | Print text to terminal |
| `nano <file>` | Open file in nano editor |
| `man <command>` | Open manual page for a command |

---

## Shell Operators

| Operator | Purpose |
|----------|---------|
| `&` | Run command in background |
| `&&` | Run second command only if first succeeds |
| `>` | Redirect output to file (overwrites) |
| `>>` | Append output to file |

**Examples:**
```bash
echo "hello" > welcome       # creates/overwrites file
echo "world" >> welcome      # appends to file
cp bigfile.zip /backup/ &    # runs copy in background
```

---

## File Permissions

Format: `rwxrwxrwx` → Owner / Group / Others

| Symbol | Value |
|--------|-------|
| r (read) | 4 |
| w (write) | 2 |
| x (execute) | 1 |

`rwxr-xr-x` = **755** → Owner: full, Group/Others: read+execute  
`rw-r--r--` = **644** → Owner: read/write, Others: read only

```bash
chmod 755 file.sh
```

---

## Users & Switching

```bash
whoami              # show current user
su user2            # switch to user2
su -l user2         # switch and load user2's environment
```

---

## Processes

```bash
ps              # show processes for current session
ps aux          # show all processes (all users)
top             # real-time process viewer
kill <PID>      # send SIGTERM (clean kill)
kill -9 <PID>   # send SIGKILL (force kill)
fg              # bring backgrounded process to foreground
Ctrl + Z        # background/pause current process
```

**Signals:**
- `SIGTERM` — clean kill, allows cleanup
- `SIGKILL` — force kill immediately
- `SIGSTOP` — suspend/pause

---

## Services (systemctl)

```bash
systemctl start apache2
systemctl stop apache2
systemctl enable apache2     # start on boot
systemctl disable apache2
systemctl status apache2
```

---

## Cron Jobs

Format: `MIN HOUR DOM MON DOW CMD`

```bash
crontab -e                   # edit your crontab
0 */12 * * * cp -R /home/user/Docs /var/backups/   # backup every 12hrs
```

Use `*` as wildcard for "any". `@reboot` runs on system boot.

---

## File Transfer

```bash
# Download from web
wget https://example.com/file.txt

# SCP — copy to remote
scp file.txt user@192.168.1.30:/home/user/file.txt

# SCP — copy from remote
scp user@192.168.1.30:/home/user/file.txt ./local_file.txt

# Spin up quick HTTP server (serves current directory on port 8000)
python3 -m http.server
```

---

## Important Directories

| Path | Purpose |
|------|---------|
| `/etc` | System config files (sudoers, passwd, shadow) |
| `/var/log` | Application and service logs |
| `/tmp` | Temporary files — cleared on reboot, world-writable |
| `/root` | Home directory for the root user |
| `/home/<user>` | Standard user home directories |

---

## SSH

```bash
ssh username@MACHINE_IP
```

Encrypted protocol for remote terminal access. Password input won't show on screen — that's normal, just type and hit Enter.

---

## Notes / Things to Add

*Use this space as you pick up new stuff:*

-
-
-

