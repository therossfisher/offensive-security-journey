# Bug Bounty Recon Automation Workflow

The bugs that pay are on assets nobody else has found yet. Top hunters win with better recon, not better exploitation skills. The goal is a continuous monitoring pipeline that alerts you the moment a new asset appears.

**Prerequisites:** Complete Jr Pen Tester path first. Understand what these tools are doing before automating them.

---

## The Core Concept

New subdomains appear constantly as companies deploy new services. The hunters who find bugs first are running automated recon 24/7 and getting notified immediately when something new shows up — before anyone else pokes at it.

Pipeline goal:
```
Find new assets → Probe them → Screenshot → Scan → Alert
```

---

## Phase 1 — Choose Your Target

Start on HackerOne or Bugcrowd. Look for programs with:
- **Broad scope** — `*.company.com` wildcard scope is ideal
- **Active program** — recent activity, paid bounties
- **Medium/large companies** — more assets = more attack surface

Resources:
- https://hackerone.com/programs
- https://bugcrowd.com/programs
- **Chaos Project** — https://chaos.projectdiscovery.io — free dataset of bug bounty program scope domains

---

## Phase 2 — Map the Target's Full Asset Footprint

Before touching subdomains, find everything the company owns.

### Step 1 — Find ASNs
```bash
# Manual — Hurricane Electric BGP Toolkit
https://bgp.he.net

# Search by company name, find all ASNs and IP ranges they own
# Automated
asnlookup -o "Tesla"
metabigor net --org "Tesla"
```

### Step 2 — Find All Related Domains via Reverse WHOIS
```bash
# Manual
https://whoxy.com — search by company name or registrant email

# Automated
domlink -t company.com -o domains.txt
```

### Step 3 — Find Acquisitions
```bash
# Manual research
# 1. Google: "company acquisitions"
# 2. Crunchbase: https://crunchbase.com/organization/company/acquisitions
# 3. Wikipedia acquisitions section
# Each acquired company = new seed domain potentially in scope
```

### Step 4 — Find Related Domains via Analytics/Ad Pixels
```bash
# BuiltWith Relationships — finds domains sharing same Google Analytics ID
https://builtwith.com/relationships/target.com

# This finds assets the company owns that aren't obviously branded
```

### Step 5 — Expand Seed Domains with Amass Intel Mode
```bash
amass intel -org "Tesla" -asn 394161 -o seed_domains.txt
amass intel -whois -d tesla.com -o more_domains.txt
```

---

## Phase 3 — Subdomain Enumeration

Run multiple tools — each pulls from different sources. More overlap = more confidence. More unique results = more coverage.

### Passive Sources (no traffic to target)
```bash
# Subfinder — aggregates 40+ passive sources
subfinder -d target.com -o subfinder_results.txt

# Amass passive mode
amass enum -passive -d target.com -o amass_results.txt

# crt.sh via curl
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u

# theHarvester
theHarvester -d target.com -b google,bing,certspotter -f harvester_results

# Assetfinder
assetfinder --subs-only target.com

# GitHub subdomain scraping
# https://github.com/gwen001/github-search
python3 github-subdomains.py -t YOUR_GITHUB_TOKEN -d target.com
```

### Combine and Deduplicate Results
```bash
cat subfinder_results.txt amass_results.txt > all_subs.txt
sort -u all_subs.txt > unique_subs.txt
```

### Subdomain Bruteforcing (semi-active)
```bash
# Massdns with SecLists wordlist
massdns -r resolvers.txt -t A -o S -w massdns_results.txt unique_subs.txt

# Amass bruteforce mode
amass enum -brute -d target.com -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

### Alteration Scanning — Find Numbered/Patterned Subdomains
```bash
# Altdns finds variations like dev1, dev2, staging1, api-v2
altdns -i unique_subs.txt -o altdns_output.txt -w words.txt -r -s resolved_altdns.txt
```

### TLS Certificate Scanning
```bash
# bufferover.run — finds domains from TLS certs on cloud IP ranges
curl -s https://dns.bufferover.run/dns?q=.target.com | jq -r '.FDNS_A[]' | cut -d ',' -f2 | sort -u
```

---

## Phase 4 — Probe What's Live

Not every subdomain has a web service. Filter to what's actually running HTTP/HTTPS.

```bash
# httpx — fast HTTP prober
cat unique_subs.txt | httpx -silent -o live_hosts.txt

# httprobe (Tom Hudson)
cat unique_subs.txt | httprobe | tee live_hosts.txt
```

---

## Phase 5 — Port Scan Live Hosts

Look beyond 80/443. Jenkins on 8080, admin panels on 8443, databases exposed, etc.

```bash
# Masscan for speed across IP ranges
sudo masscan -p1-65535 --rate=1000 -iL live_ips.txt -oG masscan_results.txt

# Nmap service detection on masscan results
nmap -sV -iL live_ips.txt -p $(cat masscan_results.txt | grep open | awk '{print $4}' | cut -d'/' -f1 | sort -u | tr '\n' ',') -oN nmap_services.txt

# Brutespray — check for default credentials on discovered services
brutespray --file nmap_services.txt --threads 5
```

---

## Phase 6 — Screenshot Everything (Domain Flyover)

Visual review of hundreds of subdomains at once. Spot login panels, admin interfaces, default pages, and forgotten apps fast.

```bash
# Aquatone
cat live_hosts.txt | aquatone -out aquatone_report/

# EyeWitness (also grabs server headers)
eyewitness --web -f live_hosts.txt -d eyewitness_report/
```

Review the screenshots manually. Look for:
- Login panels on unexpected subdomains
- Default server pages (nginx/apache default = unconfigured, possibly interesting)
- Internal-looking apps exposed externally
- Old or forgotten services

---

## Phase 7 — JavaScript Recon

Subdomains and API keys hide in JavaScript files. Crawl all live hosts and extract them.

```bash
# GoSpider — crawls HTML and JS links
gospider -S live_hosts.txt -o gospider_output/ -c 10 -d 3

# Hakrawler
cat live_hosts.txt | hakrawler -d 3 -t 20 | tee crawl_results.txt

# Subdomainizer — extracts subdomains AND flags high-entropy strings (API keys)
python3 SubDomainizer.py -u https://target.com -o subdomainizer_results.txt
```

---

## Phase 8 — GitHub Recon

Search GitHub for leaked credentials, API keys, internal subdomains, config files.

```bash
# Use jhaddix's GitHub search script
# https://gist.github.com/jhaddix/1fb7ab2409ab579178d2a79959909b33

# Manual GitHub search queries:
# "target.com" password
# "target.com" api_key
# "target.com" secret
# "target.com" internal
# org:companyname password
# org:companyname .env
```

Tools:
- **GitLeaks** — `gitleaks detect --source /path/to/repo`
- **TruffleHog** — scans git history for secrets
- **gitrob** — finds sensitive files across GitHub organizations

---

## Phase 9 — Automated Vulnerability Scanning

Run Nuclei templates against all live hosts. Fast, low-noise, covers common misconfigs and CVEs.

```bash
# Nuclei against live hosts
nuclei -l live_hosts.txt -t nuclei-templates/ -o nuclei_results.txt

# Subdomain takeover templates specifically
nuclei -l live_hosts.txt -t nuclei-templates/takeovers/ -o takeover_results.txt

# Expose panels, default creds, misconfigs
nuclei -l live_hosts.txt -tags exposure,misconfig,default-login -o exposure_results.txt
```

---

## Phase 10 — Subdomain Takeover Hunting

One of the most beginner-accessible high-value findings. Detectable with automation, doesn't require deep exploitation skills.

```bash
# Check can-i-take-over-xyz reference
https://github.com/EdOverflow/can-i-take-over-xyz

# Subjack — automated takeover scanner
subjack -w unique_subs.txt -t 100 -o subjack_results.txt -ssl

# Nuclei takeover templates (already covered in Phase 9)
```

**What you're looking for:** A subdomain with a CNAME pointing to a third-party service (Heroku, GitHub Pages, S3, Shopify, etc.) that has been deprovisioned. You can claim it.

---

## The All-in-One Option — ReconFTW

Chains most of the above into a single automated pipeline. Good for getting started and understanding what a full recon looks like.

```bash
# Install
git clone https://github.com/six2dez/reconftw
cd reconftw
./install.sh

# Full passive recon
./reconftw.sh -d target.com -r

# Recon + active scanning
./reconftw.sh -d target.com -f

# Subdomain enum only
./reconftw.sh -d target.com -s
```

**Caveat:** ReconFTW is great for learning what the pipeline looks like but understand each tool it uses before relying on its output. It can be noisy and slow on large targets.

---

## Continuous Monitoring Setup

The real edge comes from monitoring for new assets 24/7. When a new subdomain appears, you want to be first.

### Basic Monitoring Script Concept
```bash
#!/bin/bash
# Run daily via cron
TARGET="target.com"
DATE=$(date +%Y%m%d)
PREV_DATE=$(date -d "yesterday" +%Y%m%d)

# Run subfinder
subfinder -d $TARGET -silent -o /recon/$TARGET/$DATE_subs.txt

# Diff against yesterday
diff /recon/$TARGET/$PREV_DATE_subs.txt /recon/$TARGET/$DATE_subs.txt | grep "^>" | cut -d' ' -f2 > /recon/$TARGET/new_$DATE.txt

# If new subdomains found, probe and notify
if [ -s /recon/$TARGET/new_$DATE.txt ]; then
    cat /recon/$TARGET/new_$DATE.txt | httpx -silent | notify
fi
```

### notify (ProjectDiscovery)
Sends alerts to Slack, Discord, Telegram, or email when new findings appear.
```bash
# Install
go install github.com/projectdiscovery/notify/cmd/notify@latest

# Pipe any tool output to notify
subfinder -d target.com | notify
nuclei -l hosts.txt | notify
```

---

## Tool Install List (Kali)

```bash
# Go tools
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
go install github.com/projectdiscovery/notify/cmd/notify@latest
go install github.com/hakluke/hakrawler@latest
go install github.com/tomnomnom/httprobe@latest
go install github.com/tomnomnom/assetfinder@latest
go install github.com/jaeles-project/gospider@latest
go install github.com/michenriksen/aquatone@latest

# Apt
sudo apt install amass massdns altdns eyewitness -y

# Pip
pip3 install theHarvester
pip3 install SubDomainizer

# Git clones
git clone https://github.com/blechschmidt/massdns
git clone https://github.com/EdOverflow/can-i-take-over-xyz
git clone https://github.com/six2dez/reconftw
git clone https://github.com/gwen001/github-search
```

---

## Key Resources

- **Jason Haddix TBHM v4** — https://www.youtube.com/watch?v=qLTe6Z10vj8
- **Nahamsec Recon** — YouTube, Sunday Recon series
- **ProjectDiscovery** — https://github.com/projectdiscovery — makers of Subfinder, httpx, Nuclei, Notify
- **SecLists** — https://github.com/danielmiessler/SecLists — wordlists for bruteforcing
- **Chaos Project** — https://chaos.projectdiscovery.io — bug bounty scope domains dataset
- **can-i-take-over-xyz** — https://github.com/EdOverflow/can-i-take-over-xyz
- **OSINT Framework** — https://osintframework.com

---

## Where to Start When You're Ready

1. Pick one HackerOne program with `*.company.com` wildcard scope
2. Install Subfinder, httpx, Nuclei, Aquatone
3. Run manual recon first — understand what each tool returns
4. Find one new subdomain nobody has reported on
5. Screenshot it, probe it, run Nuclei against it
6. Document everything as you go
7. Only then start automating and chaining tools together
