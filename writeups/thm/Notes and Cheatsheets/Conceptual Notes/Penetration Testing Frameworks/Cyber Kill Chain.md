# Cyber Kill Chain

Introduced by Lockheed Martin in 2011. A framework that breaks a cyberattack into 7 stages to help defenders detect and interrupt attacks in progress.

---

## The 7 Stages

### 1. Reconnaissance
Attacker gathers information about the target before doing anything.

**Passive** — no direct interaction, stays invisible:
- WHOIS and DNS lookups
- Website crawling and scraping
- Google Dorking
- Social media recon

**Active** — requires interaction, makes noise:
- Network port scanning
- Vulnerability scanning
- Physical surveillance of premises

**Defense:** Minimize public info exposure, monitor network traffic for scanning activity, review service logs.

---

### 2. Weaponisation
Attacker builds or modifies a payload tailored to the target's vulnerabilities.

- Uses exploit kits, custom exploits, or modifies existing ones
- Hides payload in innocuous files (Word doc, PDF, USB)
- Uses obfuscation or encryption to evade detection
- Malicious macros in Office documents are common

**Defense:** User training, disable macros, remove unnecessary software/plugins, restrict attack surface.

---

### 3. Delivery
Attacker transmits the payload to the target.

- Phishing emails with malicious attachments
- Spear phishing — targeted, spoofed to look like trusted sender
- Malicious web links, drive-by downloads
- File sharing platforms
- Malvertising
- SMS phishing (smishing)
- Physical delivery — USB drops, malicious DVDs

**Defense:** User awareness training, email/web filtering, WAF, network monitoring, patch management.

---

### 4. Exploitation
Payload executes and exploits a vulnerability.

- Weak or default passwords
- Software vulnerabilities (CVEs)
- Zero-day exploits — no patch exists yet
- SQL injection, buffer overflow, misconfigurations

**Defense:** MFA, patch management, vulnerability scanning, IPS/IDS, WAF.

---

### 5. Installation
Attacker establishes persistence so they can return without re-exploiting.

- Scheduled tasks (Windows) or cron jobs (Linux)
- New services or daemons
- Startup script modification
- Rootkits, backdoors, malware
- Web shells on compromised web servers
- Living-off-the-land binaries (LOLBins) — using legitimate system tools

**Defense:** Monitor new processes and services, EDR, system audits vs baseline, application allowlisting.

---

### 6. Command and Control (C2)
Attacker establishes a covert communication channel to the compromised system.

- Uses common protocols to blend in: HTTP, HTTPS, DNS, SMTP
- DNS tunneling — encodes data in DNS requests
- Uses legitimate services: Dropbox, Google Docs, social media DMs
- DGA (Domain Generation Algorithms) — generates thousands of domains, registers a small percentage, malware cycles through them
- Fast Flux — hundreds of IPs mapped to one domain, rotated every few minutes using compromised devices as proxies

**Defense:** Network monitoring, DNS traffic analysis, web content filtering, encryption inspection, honeypots.

---

### 7. Actions on Objectives
Attacker executes their end goal.

- **Data exfiltration** — stealing sensitive files (espionage)
- **Ransomware** — encrypt data, demand payment
- **Destructive attacks** — delete or corrupt data
- **Financial theft** — fraudulent transactions, wire transfers
- **Lateral movement** — compromise other systems on the network
- **ICS manipulation** — target industrial control systems
- **Long-term persistence** — stay hidden for a future attack

**Defense:** DLP solutions, backup and recovery plan, network segmentation, least privilege access controls, user activity monitoring, EDR.

---

## Key Takeaways

- The kill chain is sequential — defenders can break the attack at **any stage**
- Most damage happens at stage 7, but earlier detection = less damage
- **Red teamers** follow this chain to simulate realistic attacks
- **Blue teamers** monitor for indicators at each stage to detect and block

---

## Quick Reference

| Stage | Attacker Goal | Key Defense |
|---|---|---|
| Reconnaissance | Gather intel | Minimize exposure, monitor logs |
| Weaponisation | Build payload | User training, disable macros |
| Delivery | Send payload | Email filtering, WAF |
| Exploitation | Execute payload | Patching, MFA, IPS |
| Installation | Gain persistence | EDR, process monitoring |
| C2 | Establish control | Network monitoring, DNS analysis |
| Actions on Objectives | Achieve goal | DLP, segmentation, backups |
