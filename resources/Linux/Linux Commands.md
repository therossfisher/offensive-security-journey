
Linux Stuff Quick reference

---

## System Info

```bash
whoami                          # current user
id                              # user ID, group memberships
hostname                        # machine name
uname -a                        # kernel version and OS info
df -h                           # disk space usage
du -sh /*                       # find what's eating disk space
lsblk                           # list block devices and partitions
ip a                            # show all network interfaces
ip a | grep tun0                # check VPN interface
ping -c 3 TARGET                # test connectivity
```

---

## File System

```bash
ls -la                          # list files including hidden, with permissions
find /home/USER -name "*.md"    # find files by name
find / -name "*.ovpn" 2>/dev/null  # find files, suppress errors
cat FILE                        # print file contents
less FILE                       # scroll through file
head -c 500 FILE                # first 500 characters of file
grep -i "keyword" FILE          # search file for keyword (case insensitive)
grep -r "keyword" /path/        # recursive search through directory
grep -oP 'pattern' FILE         # extract matches using regex
mkdir -p nmap/                  # create directory (and parents if needed)
cp FILE /destination/           # copy file
mv FILE /destination/           # move or rename file
rm FILE                         # delete file
chmod +x FILE                   # make file executable
chmod -R 755 /directory/        # recursively set permissions
chown USER:USER FILE             # change file owner
sudo chown -R USER:USER /dir/   # recursively change owner
```

---

## Process Management

```bash
ps aux | grep openvpn           # find a running process
sudo pkill openvpn              # kill process by name
sudo kill PID                   # kill process by ID
Ctrl+C                          # stop current running command
Ctrl+Z                          # suspend current command
jobs                            # list background jobs
```

---

## Networking

```bash
ip a                            # all interfaces
ip a | grep tun0                # VPN interface check
ping -c 3 TARGET            # test connectivity
nc -lvnp 443                    # start netcat listener on port 443
sudo pkill openvpn              # kill VPN connection
sudo openvpn FILE.ovpn          # connect VPN
```

---
```bash
# When a page redirects to a hostname that doesn't resolve
# Add entry
echo "TARGET_IP hostname.htb" | sudo tee -a /etc/hosts

# Verify it worked
cat /etc/hosts | tail -5

# Remove it when done
sudo nano /etc/hosts
# Find the line and delete it
```
---
## Package Management

```bash
sudo apt update                 # update package list
sudo apt install PACKAGE -y     # install package
sudo apt install PACKAGE --break-system-packages  # for pip installs
sudo apt clean                  # clear apt cache
sudo apt autoremove -y          # remove unused packages
pip3 install requests --break-system-packages     # install python package
```

---

## Shell Configuration

```bash
nano ~/.zshrc                   # edit shell config
source ~/.zshrc                 # reload shell config without restarting
alias                           # list all current aliases
echo $DISPLAY                   # check display variable (for GUI apps)
```

**Useful aliases in ~/.zshrc:**
```bash
alias htb='sudo openvpn ~/Downloads/htb.ovpn'
alias htbsp='sudo openvpn ~/Downloads/htb_sp.ovpn'
alias obsidian='/opt/squashfs-root/obsidian --no-sandbox --disable-gpu'
alias thm='sudo openvpn ~/Downloads/thm.ovpn'
```

---

## Curl

```bash
curl -I http://TARGET            # headers only (HEAD request)
curl -s http://TARGET            # silent, full response body
curl -v http://TARGET            # verbose, headers + body + request
curl -sI http://TARGET | grep -i server    # fingerprint server
curl -c cookies.txt http://TARGET          # save cookies
curl -b cookies.txt http://TARGET          # send saved cookies
curl -X PUT http://TARGET        # use PUT method
curl -X OPTIONS http://TARGET -sv 2>&1 | grep "Allow:"  # check methods
curl -T FILE http://TARGET/path/ # upload file via PUT
curl --ntlm -u 'user:pass' -T FILE http://TARGET/  # NTLM auth upload
curl -g -G "http://TARGET/page"  # disable glob parsing (for square brackets)
curl --data-urlencode "cmd@file.txt" http://TARGET  # send file contents URL-encoded
curl --max-time 10 http://TARGET # timeout after 10 seconds
curl -s http://TARGET | grep -i "keyword"  # grep response body
curl -s http://TARGET | python3 -m json.tool  # pretty print JSON response
curl -s http://TARGET | wc -c   # count response bytes
```

---

## Nmap

```bash
# Basic scans
nmap -p- --min-rate 5000 -Pn TARGET         # all ports, fast, skip ping
nmap -sV -sC --reason -oA nmap/facts TARGET  # version + scripts + save output
nmap -sV -p 22,80,443,3000,8080 TARGET       # specific ports
nmap -sU --top-ports 100 -Pn TARGET          # UDP top 100 ports

# NSE scripts
nmap --script http-methods -p 80 TARGET
nmap --script http-webdav-scan -p 80 TARGET
nmap --script http-ntlm-info --script-args http-ntlm-info.root=/webdav/ -p 80 TARGET

# Flags reference
# -p-        all 65535 ports
# -Pn        skip host discovery (treat as up)
# -sV        version detection
# -sC        default scripts
# -sS        SYN scan (requires sudo)
# -sU        UDP scan
# --reason   show why port is open/closed/filtered
# --min-rate speed up scan
# -oA        output all formats (xml, nmap, gnmap)
```

---

## Gobuster

```bash
# Directory enumeration
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://TARGET -w /usr/share/seclists/Discovery/Web-Content/common.txt
gobuster dir -u http://TARGET -w WORDLIST -x php,html,txt,bak -t 20
gobuster dir -u http://TARGET -w WORDLIST -s 307,302 -b ""  # find redirects only

# DNS subdomain enumeration
gobuster dns -d TARGET.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Flags reference
# -u    target URL
# -w    wordlist
# -x    file extensions to append
# -t    threads
# -s    only show these status codes
# -b    blacklist status codes (use "" to clear default)
# -r    follow redirects
# -o    output to file
```

---

## ffuf

```bash
ffuf -u http://TARGET/FUZZ -w WORDLIST
ffuf -u http://TARGET/FUZZ -w WORDLIST -e .php,.html,.txt
ffuf -u "http://TARGET/page?FUZZ=test" -w WORDLIST  # parameter fuzzing
ffuf -u http://TARGET/FUZZ -w WORDLIST -fs 4242      # filter by size
ffuf -u http://TARGET/FUZZ -w WORDLIST -mc 200,301,302,307  # match codes
```

---

## Nikto

```bash
nikto -h http://TARGET
nikto -h http://TARGET -nointeractive
nikto -h http://TARGET -Tuning b        # software ID only (fastest)
nikto -h http://TARGET -o results.txt   # save output
nikto -h http://TARGET -useproxy http://127.0.0.1:8080  # route through Caido
```

---

## Hydra

```bash
# SSH brute force
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt TARGET ssh -t 4 -f -v

# HTTP form
hydra -l admin -P /usr/share/wordlists/rockyou.txt TARGET http-post-form "/login:username=^USER^&password=^PASS^:Invalid"

# Flags reference
# -l    single username
# -L    username list file
# -p    single password
# -P    password list file
# -t    threads
# -f    stop on first valid credential
# -v    verbose
```

---

## SSH

```bash
ssh user@TARGET                  # connect
ssh user@TARGET -p 2222          # non-standard port
ssh -i key.pem user@TARGET       # connect with key file
```

---

## Wordlists on Kali

```bash
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## Pipes and Redirection

```bash
command | grep "keyword"         # filter output
command | sort -u                # sort and deduplicate
command | head -20               # first 20 lines
command | wc -c                  # count characters
command | wc -l                  # count lines
command 2>/dev/null              # suppress errors
command > file.txt               # save output to file
command >> file.txt              # append output to file
command1 && command2             # run command2 only if command1 succeeds
command1 || command2             # run command2 only if command1 fails
diff file1 file2                 # compare two files
```

---

## For Loops

```bash
# Check multiple ports
for port in 80 8000 3000 8080; do
    echo "=== Port $port ==="
    curl -sI http://TARGET:$port/ | grep -i server
done

# Run command against list of targets
for ip in $(cat targets.txt); do
    echo "=== $ip ==="
    nmap -p 80,443 $ip
done
```

---

## Git / GitHub

```bash
git add .
git commit -m "message"
git push
gitsave "message"               # custom alias: add + commit + push in one
```

---

## Python

```bash
python3 script.py               # run script
python3 -m json.tool            # pretty print JSON (pipe into this)
python3 -m http.server 8080     # start simple HTTP server in current directory
pip3 install requests --break-system-packages
```

---

## Kali-Specific

```bash
# Obsidian
obsidian                        # launch (alias)
/opt/squashfs-root/obsidian --no-sandbox --disable-gpu  # full command

# VPN
htb                             # connect HTB machines VPN
htbsp                           # connect HTB starting point VPN
thm                             # connect THM VPN
sudo pkill openvpn              # disconnect any VPN

# Disk
df -h                           # check free space
sudo growpart /dev/sda 1        # expand partition after VM resize
sudo resize2fs /dev/sda1        # resize filesystem after growpart
```
