# Passive Reconnaissance

Gathering intelligence from public sources without sending any traffic to the target. No direct interaction — fully stealthy, no alerts triggered, minimal legal risk.

**Key distinction:**
- **Passive** — public data only, no contact with target (WHOIS, DNS, CT logs, Shodan)
- **Active** — direct interaction required, detectable (port scanning, fuzzing, social engineering)

Note: Any direct interaction with a person affiliated with the target counts as active recon even without sending packets.

## Recon Organization Tool

**XMind** — mind mapping tool used by Jason Haddix (TBHM) to organize recon findings in a tree structure.

- Free tier available
- Cross-platform
- https://xmind.app
- Use to visually map target attack surface as you enumerate — subdomains, IPs, services, findings

---

## WHOIS

Query domain registration details.

```bash
whois tryhackme.com
```

**What you get:**
- Registrar (GoDaddy, Namecheap, etc.)
- Creation, updated, and expiration dates
- Name servers — potential targets
- Status codes (e.g. `clientTransferProhibited` = locked against hijacking)
- Abuse contacts

**Reality check:** Personal details are usually redacted since GDPR (2018). Focus on dates, registrar, and name servers.

**WHOIS is being replaced by RDAP** (as of Jan 2025 for gTLDs):
```bash
curl -s https://rdap.verisign.com/com/v1/domain/tryhackme.com | jq .
```
RDAP uses HTTPS, returns structured JSON, better privacy controls.

**Historical WHOIS:** https://www.whoxy.com — reveals previous owners, registrar changes, name server migrations.

---

## DNS Lookups

### Record Types

| Type | Returns |
|---|---|
| `A` | IPv4 address |
| `AAAA` | IPv6 address |
| `CNAME` | Alias pointing one domain to another |
| `MX` | Mail servers (with priority numbers — lower = higher priority) |
| `TXT` | SPF, DKIM, DMARC, domain verification strings |
| `SOA` | Primary name server, admin email, zone serial |
| `NS` | Authoritative name servers |

### dig (preferred)
```bash
dig tryhackme.com A
dig @1.1.1.1 tryhackme.com MX
dig tryhackme.com TXT
dig tryhackme.com ANY
```

### nslookup (legacy, Windows compatible)
```bash
nslookup -type=A tryhackme.com
nslookup -type=MX tryhackme.com 1.1.1.1
nslookup -type=TXT tryhackme.com
```

**Privacy tip:** Use `1.1.1.1` (Cloudflare) or `8.8.8.8` (Google) as resolver to avoid ISP logging your queries.

---

## Subdomain Enumeration (Passive)

Standard DNS lookups only resolve names you already know. These methods find unadvertised subdomains.

### crt.sh — Certificate Transparency Logs (best method)
Every SSL/TLS cert issued since ~2015 is publicly logged. Each cert contains SANs (Subject Alternative Names) listing all covered subdomains.

```
https://crt.sh/?q=%.tryhackme.com
```
- Fully passive, real-time, no rate limits
- Often reveals 10-100x more subdomains than other methods

### DNSDumpster
```
https://dnsdumpster.com
```
- Aggregates public DNS data — search engine caches, zone transfer databases, CT records
- Provides visual graph of subdomains, IPs, MX records and relationships
- Does not brute-force — fully passive

### Other Passive Subdomain Tools
- **Subfinder** — CLI tool aggregating multiple passive sources
- **Amass** — OSINT framework for attack surface mapping
- **SecurityTrails** — https://securitytrails.com (free limited searches)
- **VirusTotal** — https://www.virustotal.com (subdomains tab)
- **RapidDNS** — https://rapiddns.io

---

## Shodan

Search engine for internet-connected devices. Continuously scans the public internet and indexes banners, open ports, and service responses.

```
https://www.shodan.io
```

### Search Filters
```
hostname:tryhackme.com
org:"TryHackMe"
port:443 country:US
http.component:"wordpress"
ip:104.26.10.229
```

### What You Get Per Host
- IP address and ASN
- Hosting provider/organisation
- Geographic location
- Open ports and service banners with version strings
- Tags (cdn, vuln, etc.)

### Similar Tools
- **Censys** — https://search.censys.io — host and certificate data, good for cross-referencing
- **ZoomEye** — Chinese equivalent of Shodan
- **FOFA** — another internet asset search engine

---

## OSINT and Additional Passive Recon Tools

### Email and Employee Intelligence
- **LinkedIn** — employee names, job titles, tech stack from job postings
- **Hunter.io** — https://hunter.io — find email addresses for a domain
- **Phonebook.cz** — email, domain, and URL lookup
- **theHarvester** — CLI tool harvesting emails, subdomains, IPs from public sources
```bash
theHarvester -d tryhackme.com -b google
```

### Google Dorking
Using Google search operators to find sensitive exposed data:

| Dork | Finds |
|---|---|
| `site:tryhackme.com` | All indexed pages for a domain |
| `site:tryhackme.com filetype:pdf` | PDFs on the domain |
| `intitle:"index of"` | Open directory listings |
| `inurl:admin site:tryhackme.com` | Admin pages |
| `site:tryhackme.com -www` | Subdomains (excludes www) |

### GitHub Recon
Search public repos for hardcoded credentials, API keys, config files:
- Search GitHub for `org:companyname password` or `org:companyname apikey`
- **GitLeaks** — automated scanning for secrets in git repos
- **TruffleHog** — searches git history for high-entropy strings

### Web Archive / Historical Data
- **Wayback Machine** — https://web.archive.org — historical snapshots of websites
- **CachedView** — Google cache of pages

### Breach Data
- **HaveIBeenPwned** — https://haveibeenpwned.com — check if emails appear in breaches
- **DeHashed** — https://dehashed.com — search breach data (subscription)

### Network and IP Intelligence
- **BGP Toolkit** — https://bgp.he.net — ASN lookup, IP ranges, BGP routing info
- **IPinfo** — https://ipinfo.io — IP geolocation, ASN, hosting provider
- **BuiltWith** — https://builtwith.com — technology stack detection from public sources
- **Wappalyzer** — browser extension for tech stack fingerprinting

### Comprehensive OSINT Frameworks
- **OSINT Framework** — https://osintframework.com — visual map of all OSINT tools by category
- **Maltego** — graphical link analysis tool for OSINT investigations
- **Recon-ng** — modular CLI recon framework (like Metasploit for OSINT)
- **SpiderFoot** — automated OSINT collection across 200+ sources

---

## Command Quick Reference

| Purpose | Command |
|---|---|
| WHOIS lookup | `whois tryhackme.com` |
| RDAP lookup | `curl -s https://rdap.verisign.com/com/v1/domain/DOMAIN.com \| jq .` |
| DNS A records | `dig tryhackme.com A` |
| DNS MX records | `dig @1.1.1.1 tryhackme.com MX` |
| DNS TXT records | `dig tryhackme.com TXT` |
| All DNS records | `dig tryhackme.com ANY` |
| Email harvesting | `theHarvester -d tryhackme.com -b google` |
| CT subdomain search | `https://crt.sh/?q=%.tryhackme.com` |
| Shodan search | `https://shodan.io` → `hostname:domain.com` |

---

## Defender Tips
- Monitor your own footprint — set Shodan/Censys alerts for your IP ranges
- Watch CT logs for new certificates on your domains (subdomain takeover risk)
- Track DNS changes — unexpected MX or TXT records can signal compromise
- Minimize public exposure — remove sensitive info from WHOIS, job postings, GitHub repos
- Check HaveIBeenPwned for your domain's email addresses regularly

## Jason Haddix TBHM v4 — Additional Passive Recon Tools

### ASN and IP Range Discovery
- **BGP Toolkit** — https://bgp.he.net — find ASNs belonging to a company (already in notes)
- **ASNlookup** — https://github.com/yassineaboukir/Asnlookup — automate ASN discovery
- **metabigor** — https://github.com/j3ssie/metabigor — ASN and IP range enumeration
- **Amass (intel mode)** — discover seed domains from ASNs

### Reverse WHOIS / Domain Discovery
- **Whoxy** — https://whoxy.com — reverse WHOIS to find all domains registered to a company
- **DOMLink** — https://github.com/vysecurity/DomLink — queries Whoxy API automatically

### Ad/Analytics Pixel Tracking
- **BuiltWith Relationships** — https://builtwith.com/relationships/domain.com — finds related domains sharing the same tracking/analytics IDs (Google Analytics, ad pixels)

### JavaScript and Link Crawling
- **GoSpider** — https://github.com/jaeles-project/gospider — crawls sites extracting links from HTML and JS
- **Hakrawler** — https://github.com/hakluke/hakrawler — fast web crawler for recon
- **Subdomainizer** — finds subdomains in JS, computes Shannon entropy to flag API keys and passwords, finds S3 buckets

### Subdomain Bruteforcing and Alteration
- **Massdns** — fast DNS bruteforcer
- **Altdns** — https://github.com/infosec-au/altdns — alteration scanning (finds dev1, dev2, staging1 patterns)

### TLS Certificate Scanning
- **tls.bufferover.run** — API that monitors cloud IP ranges for HTTPS servers and extracts domain names from their certs — fully passive

### GitHub Recon
- **github-search** — https://github.com/gwen001/github-search — scrape GitHub API for subdomains and sensitive data
- **shosubgo** — find subdomains via Shodan API

### Screenshot / Domain Flyover
- **Aquatone** — screenshot tool for large numbers of domains
- **EyeWitness** — screenshots plus server headers and additional intel
- **HTTPScreenshot** — another screenshot tool

### Subdomain Takeover
- **can-i-take-over-xyz** — https://github.com/EdOverflow/can-i-take-over-xyz — reference for which services are vulnerable to subdomain takeover
- **Nuclei** — has templates to automate subdomain takeover detection

### Automation and Orchestration
- **Interlace** — https://github.com/codingo/Interlace — wraps single-threaded tools to make them multithreaded for large-scale recon

### Acquisitions Research
- **Crunchbase** — https://crunchbase.com — find company acquisitions
- Wikipedia acquisition pages
