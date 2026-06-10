# Content Discovery

**THM Room:** Content Discovery  
**Path:** Jr Penetration Tester / Web Application Security  
**Completed:** June 2026

---

## What Is Content Discovery?

Finding content that wasn't meant to be publicly accessible or isn't linked anywhere obvious. This includes staff portals, older site versions, exposed backup files, admin panels, configuration files, and API endpoints.

**Three approaches:**
1. Manual — check known files and headers
2. OSINT — external tools and public data sources
3. Automated — tools like Gobuster with wordlists

---

## Manual Discovery

### robots.txt
Tells search engine crawlers which pages NOT to index. Site owners list sensitive directories here to keep them out of search results — which makes it a ready-made list of interesting locations.

```
http://TARGET/robots.txt
```

Key thing: robots.txt is a guideline for bots, not a security control. Listed paths are still directly accessible.

### sitemap.xml
Tells search engines which pages the owner WANTS listed. Sometimes includes staging pages, old content, or hard-to-reach URLs.

```
http://TARGET/sitemap.xml
```

Look for: sensitive paths, login pages, parameters like `?id=` that hint at injection points.

### HTTP Headers
Response headers can reveal web server software, framework, and version info. Run:

```bash
curl http://TARGET -v
```

Look for:
- `Server:` — web server software and version
- `X-Powered-By:` — framework or language
- Custom headers — sometimes contain flags, tokens, or debug info
- HTML comments — sometimes reveal hidden paths or framework info

### Favicon Fingerprinting
Favicons are often framework defaults. Hash the favicon and look it up in a database to identify the framework.

```bash
curl http://TARGET/favicon.ico | md5sum
```

Compare hash against: https://wiki.owasp.org/index.php/OWASP_favicon_database

### Framework Stack Analysis
Once framework is identified — check the framework's own documentation for:
- Default directory structures
- Admin panel paths
- Default credentials (admin/admin is still common)

### Other Manual Checks
```
http://TARGET/.env              # Environment variables, credentials
http://TARGET/web.config        # IIS configuration
http://TARGET/config.php        # PHP configuration
http://TARGET/backup.zip        # Backup archives
http://TARGET/.git/             # Exposed git repository
http://TARGET/admin             # Admin panel
http://TARGET/login             # Login page
http://TARGET/api               # API endpoints
http://TARGET/swagger           # API documentation
http://TARGET/phpinfo.php       # PHP info page
```

---

## OSINT

### Google Dorking
Advanced search operators to surface sensitive content indexed from your target.

| Filter | Example | Description |
|---|---|---|
| `site:` | `site:target.com` | Results only from specified domain |
| `inurl:` | `inurl:admin` | Results with word in URL |
| `filetype:` | `filetype:pdf` | Results of specific file type |
| `intitle:` | `intitle:admin` | Results with word in page title |
| `intext:` | `intext:password` | Results with word in body |
| `cache:` | `cache:target.com` | Google's cached version |

**Useful combinations:**
```
site:target.com filetype:pdf
site:target.com inurl:admin
site:target.com inurl:login
site:target.com filetype:env
site:target.com filetype:sql
site:target.com "index of"
```

### Wappalyzer
Browser extension and online tool that identifies technologies a website uses — frameworks, CMS, CDNs, analytics, payment processors, version numbers.
- Install as browser extension or use: https://www.wappalyzer.com

### Wayback Machine
Internet archive dating back to late 1990s. Find removed pages, old login forms, forgotten API endpoints, content published briefly then taken down.
- https://web.archive.org

### GitHub Reconnaissance
Search GitHub for company name or domain. Developers accidentally commit:
- API keys and credentials
- Configuration files
- .env files
- Internal documentation

**Important:** Check commit history, not just current files. Sensitive data removed in a later commit still exists in history.

### S3 Bucket Enumeration
Amazon S3 buckets URL format: `https://{name}.s3.amazonaws.com`

Common naming patterns to try:
```
https://company-assets.s3.amazonaws.com
https://company-backup.s3.amazonaws.com
https://company-www.s3.amazonaws.com
https://company-dev.s3.amazonaws.com
https://company-staging.s3.amazonaws.com
```

Also find bucket URLs in page source or GitHub repos.

---

## Automated Discovery — Gobuster

### Global Flags
| Flag | Description |
|---|---|
| `-t / --threads` | Concurrent threads (default 10) |
| `-w / --wordlist` | Path to wordlist — required |
| `-o / --output` | Write results to file |
| `--delay` | Wait time between requests |

### dir Mode — Directory/File Enumeration

```bash
# Basic scan
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt

# With SecLists (better)
gobuster dir -u http://TARGET -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt

# With file extensions
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -x .php,.txt,.js,.html,.bak

# Follow redirects, skip TLS verification
gobuster dir -u https://TARGET -w /usr/share/wordlists/dirb/common.txt -r -k

# Only show specific status codes
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirb/common.txt -s 200,301,302
```

**Useful dir flags:**
| Flag | Description |
|---|---|
| `-x` | File extensions to search |
| `-r` | Follow redirects |
| `-k` | Skip TLS verification |
| `-s` | Only show specific status codes |

### dns Mode — Subdomain Enumeration

```bash
gobuster dns -d example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --wildcard
```

**Useful dns flags:**
| Flag | Description |
|---|---|
| `-d` | Target domain (required) |
| `-i` | Show IP addresses subdomains resolve to |
| `-r` | Use custom DNS server |

### vhost Mode — Virtual Host Enumeration

Sends HTTP requests cycling through wordlist entries as Host: header values. Finds virtual hosts not in public DNS.

```bash
gobuster vhost -u "http://TARGET" --domain example.com -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt --append-domain --exclude-length 250-320
```

---

## Additional Techniques Not Covered in the Room

### ffuf — Faster Alternative to Gobuster
ffuf (Fuzz Faster U Fool) is faster and more flexible than Gobuster for many scenarios.

```bash
# Directory fuzzing
ffuf -u http://TARGET/FUZZ -w /usr/share/wordlists/dirb/common.txt

# File extension fuzzing
ffuf -u http://TARGET/FUZZ -w wordlist.txt -e .php,.html,.txt,.bak

# Parameter fuzzing
ffuf -u "http://TARGET/page?FUZZ=value" -w /usr/share/wordlists/dirb/common.txt

# Filter by response size to remove false positives
ffuf -u http://TARGET/FUZZ -w wordlist.txt -fs 4242
```

### feroxbuster — Recursive Directory Discovery
feroxbuster automatically recurses into discovered directories — Gobuster doesn't do this by default.

```bash
feroxbuster -u http://TARGET -w /usr/share/wordlists/dirb/common.txt
feroxbuster -u http://TARGET -w wordlist.txt -x php,html,txt
```

### dirsearch
Another directory brute-forcing tool with built-in wordlists and recursive scanning.

```bash
dirsearch -u http://TARGET
dirsearch -u http://TARGET -e php,html,js
```

### Nikto — Web Vulnerability Scanner
Scans for outdated software, dangerous files, misconfigurations automatically.

```bash
nikto -h http://TARGET
```

### WhatWeb
More detailed than Wappalyzer — identifies technologies from command line.

```bash
whatweb http://TARGET
whatweb -a 3 http://TARGET  # aggressive mode
```

### Shodan
Search engine for internet-connected devices. Find exposed services, versions, and vulnerabilities without touching the target.
- https://shodan.io
- Search: `hostname:target.com` or `org:"Company Name"`

### Certificate Transparency Logs
Find subdomains by searching SSL certificate records — no scanning required.
- https://crt.sh — search `%.target.com`
- https://censys.io

```bash
# Using curl
curl "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u
```

### theHarvester
OSINT tool that collects emails, subdomains, hosts, and names from public sources.

```bash
theHarvester -d target.com -b google,bing,linkedin,shodan
```

### Amass — Subdomain Enumeration
Most comprehensive subdomain enumeration tool — uses OSINT, brute force, and DNS.

```bash
amass enum -d target.com
amass enum -d target.com -brute -w wordlist.txt
```

### JS File Analysis
JavaScript files often contain hardcoded API endpoints, credentials, and internal paths that aren't visible in HTML.

```bash
# Download and search JS files
curl -s http://TARGET/app.js | grep -i "api\|endpoint\|password\|secret\|token\|key"

# linkfinder — extracts endpoints from JS files
python linkfinder.py -i http://TARGET -d
```

### Parameter Discovery
Finding hidden parameters on known endpoints.

```bash
# arjun — HTTP parameter discovery
arjun -u http://TARGET/api/endpoint

# ffuf for parameter fuzzing
ffuf -u "http://TARGET/page?FUZZ=test" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt
```

---

## Content Discovery Workflow

```
1. Manual checks first (quick wins)
   - robots.txt
   - sitemap.xml
   - curl -v for headers
   - View page source for comments
   - Check favicon hash

2. OSINT
   - Google dorks
   - Wappalyzer / WhatWeb
   - crt.sh for subdomains
   - GitHub search
   - Wayback Machine
   - Shodan

3. Automated scanning
   - Gobuster/ffuf dir mode with common.txt first
   - Then medium wordlist if needed
   - DNS/vhost mode for subdomains
   - Nikto for quick vuln scan
   - JS file analysis

4. Feed findings into exploitation phase
```

---

## Key Wordlists

| Wordlist | Location | Best For |
|---|---|---|
| common.txt | `/usr/share/wordlists/dirb/common.txt` | Quick dir scan |
| directory-list-2.3-medium.txt | `/usr/share/wordlists/dirbuster/` | Thorough dir scan |
| common.txt (SecLists) | `/usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt` | Best general purpose |
| subdomains-top1million-5000.txt | `/usr/share/wordlists/SecLists/Discovery/DNS/` | Subdomain enum |
| burp-parameter-names.txt | `/usr/share/wordlists/SecLists/Discovery/Web-Content/` | Parameter fuzzing |
