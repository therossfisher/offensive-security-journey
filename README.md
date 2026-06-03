# Offensive Security Learning Journey

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Profile-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/p/dreamfps)
[![GitHub followers](https://img.shields.io/github/followers/therossfisher?style=for-the-badge)](https://github.com/therossfisher)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/rossfish)

> Transitioning from network engineering into offensive security and AI red teaming. Documenting methodology, findings, and progress publicly. Networking background A.S. Computer Network Engineering | CCNA | NRS-I | Wireshark | Building toward eJPT → PNPT → OSCP.

---

## About This Repository

This is a living portfolio of my offensive security learning journey. Every room, machine, and AI red team experiment I complete gets documented here with methodology — not just "I did it" but how I thought about it, what I tried, what worked, and what I learned.

**Background:** Network engineering and IT infrastructure  
**Current focus:** Penetration testing fundamentals + AI red teaming  
**Certifications in progress:** eJPT (target: Month 6)  
**Tools:** Nmap, Burp Suite, Metasploit, Wireshark, BloodHound, PyRIT, Garak  

---

## Progress Tracker

### Certification Roadmap

| Certification | Status | Target Date | Notes |
|---|---|---|---|
| Wireshark (Course) | ✅ Complete | — | Network traffic analysis |
| TryHackMe Jr Pen Tester | 🔄 In Progress | Month 3 | |
| TryHackMe Red Teaming | ⬜ Not Started | Month 3 | |
| eJPT (INE) | ⬜ Not Started | Month 6 | |
| PNPT (TCM Security) | ⬜ Not Started | Month 9-12 | |
| OSCP | ⬜ Not Started | Month 12-18 | |

### Platform Progress

| Platform | Status | Rooms/Machines | Notes |
|---|---|---|---|
| TryHackMe | 🔄 Active | 0 | Jr Pen Tester path |
| HackTheBox | ⬜ Not Started | 0 | Start after THM |
| PortSwigger Web Academy | 🔄 Active | 0 | Running parallel |
| OWASP LLM Top 10 | 🔄 Studying | 0/10 | AI red team foundation |

---

## Repository Structure

```
pentest-journey/
│
├── phase-0-foundation/       # PortSwigger + OWASP LLM Top 10 (Free, ongoing)
├── phase-1-thm/              # TryHackMe paths and notes
├── phase-2-htb-ejpt/         # HackTheBox machines + eJPT prep
├── phase-3-intermediate/     # HTB medium + Active Directory + AI specialization
├── phase-4-pnpt-oscp/        # PNPT and OSCP preparation
│
├── writeups/
│   ├── thm/                  # TryHackMe room writeups
│   └── htb/                  # HackTheBox machine writeups
│
├── ai-redteam/
│   └── findings/             # Documented AI red team experiments
│
└── resources/                # Useful references, cheatsheets, methodology notes
```

---

## Phase Overview

### [Phase 0 — Free Foundation](./phase-0-foundation/) `ACTIVE`
PortSwigger Web Academy and OWASP LLM Top 10. Runs in parallel with all other phases. Zero cost. Building web application testing fundamentals and AI red team knowledge base simultaneously.

### [Phase 1 — TryHackMe](./phase-1-thm/) `ACTIVE`
Jr Penetration Tester path followed by Red Teaming path. Guided, structured, methodology-focused. Paired with TCM Security Practical Ethical Hacking course. PyRIT and Garak experimentation begins here.

### [Phase 2 — HackTheBox + eJPT](./phase-2-htb-ejpt/) `UPCOMING`
Unguided machines proving independent capability. INE PTS course and eJPT certification. First formal credential. AI red team portfolio building alongside.

### [Phase 3 — Intermediate](./phase-3-intermediate/) `UPCOMING`
HackTheBox medium machines. Active Directory attack chains. AI red teaming deepened into agentic systems. PNPT preparation begins.

### [Phase 4 — PNPT + OSCP](./phase-4-pnpt-oscp/) `UPCOMING`
PNPT exam. OSCP — ideally while employed in a security role. The certifications that open senior doors.

---

## AI Red Teaming Track

Running parallel to the traditional pen testing track. Documenting LLM vulnerability research mapped to OWASP LLM Top 10 and MITRE ATLAS frameworks.

| Category | OWASP LLM | Status | Findings |
|---|---|---|---|
| Prompt Injection | LLM01 | 🔄 Studying | [View](./ai-redteam/findings/) |
| Insecure Output Handling | LLM02 | ⬜ Not Started | — |
| Training Data Poisoning | LLM03 | ⬜ Not Started | — |
| Model Denial of Service | LLM04 | ⬜ Not Started | — |
| Supply Chain Vulnerabilities | LLM05 | ⬜ Not Started | — |
| Sensitive Info Disclosure | LLM06 | ⬜ Not Started | — |
| Insecure Plugin Design | LLM07 | ⬜ Not Started | — |
| Excessive Agency | LLM08 | ⬜ Not Started | — |
| Overreliance | LLM09 | ⬜ Not Started | — |
| Model Theft | LLM10 | ⬜ Not Started | — |

---

## Skills

### Technical Skills
`Network Enumeration` `Wireshark` `Nmap` `Burp Suite` `Metasploit` `Privilege Escalation` `Web Application Testing` `Prompt Injection` `OWASP LLM Top 10` `PyRIT` `Garak`

### Methodology
`Penetration Test Methodology` `Reconnaissance` `Enumeration` `Exploitation` `Post-Exploitation` `Findings Documentation` `AI Red Team Methodology` `MITRE ATLAS`

### Networking Background
`TCP/IP` `Network Architecture` `Traffic Analysis` `Packet Capture` `Network Infrastructure`

---

## Writeup Index

### TryHackMe
*Writeups added as rooms are completed*

| Room | Difficulty | Date | Key Techniques |
|---|---|---|---|
| — | — | — | — |

### HackTheBox
*Writeups added after machine retirement or with permission*

| Machine | OS | Difficulty | Date | Key Techniques |
|---|---|---|---|---|
| — | — | — | — | — |

---

## Methodology Reference

My standard engagement methodology (adapting as I learn):

```
1. RECONNAISSANCE
   └── Passive: OSINT, DNS, public info
   └── Active: Nmap scans, service enumeration

2. ENUMERATION
   └── Service-specific enumeration
   └── Web: directory busting, parameter discovery
   └── SMB, FTP, SSH fingerprinting

3. EXPLOITATION
   └── Vulnerability identification
   └── Exploit selection and execution
   └── Initial access

4. POST-EXPLOITATION
   └── Privilege escalation
   └── Lateral movement
   └── Persistence (lab environments only)

5. REPORTING
   └── Finding documentation
   └── Impact assessment
   └── Remediation recommendations
```

---

*Last updated: June 2026*
