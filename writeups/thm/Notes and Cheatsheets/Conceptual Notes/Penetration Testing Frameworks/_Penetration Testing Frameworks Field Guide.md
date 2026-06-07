# Penetration Testing Frameworks — Field Guide

A quick reference for every framework covered. Use this to identify which framework applies to a given engagement before diving into the detailed notes.

---

## At a Glance

| Framework | Type | Best For | Maintained |
|---|---|---|---|
| PTES | Pentest methodology | Standard corporate network pentests | Partially |
| OSSTMM | Scientific/quantitative | Repeatable, auditable measurements | Yes |
| OWASP WSTG | Web app testing guide | Web application pentests | Yes |
| OWASP MASTG | Mobile app testing guide | Android/iOS app pentests | Yes |
| NIST SP 800-115 | Government guidance | Federal/government assessments | Yes |
| ISSAF | Legacy methodology | Learning adversarial logic (not for active use) | No |
| MITRE ATT&CK | Adversary knowledge base | Enriching reports, threat mapping, detection | Yes |
| Cyber Kill Chain | Attack framework | Understanding attack progression | Yes |
| WASC | Web threat taxonomy | Legacy reference only | No |
| CSA CCM | Cloud governance | Cloud security posture assessments | Yes |
| PCI DSS Guidelines | Regulatory mandate | Any engagement involving payment card data | Yes |
| CBEST | Threat-intel-led pentest | UK financial institutions | Yes |

---

## The Frameworks

### PTES — Penetration Testing Execution Standard
**What it is:** End-to-end penetration testing methodology covering the full engagement lifecycle from first client call to final report.

**7 phases:** Pre-Engagement → Intelligence Gathering → Threat Modeling → Vulnerability Analysis → Exploitation → Post-Exploitation → Reporting

**Use it when:** Standard corporate network or web application pentest where you need a clear workflow and structured reporting for both technical and executive audiences.

**Why it matters:** Most closely mirrors how real engagements actually unfold. Excellent for building workflow instincts. Detailed pre-engagement guidance prevents the legal and scoping disputes that trip up inexperienced testers.

**Limitation:** Not formally updated in years — tool guidance is outdated. Supplement with current documentation.

→ [[PTES]]

---

### OSSTMM — Open Source Security Testing Methodology Manual
**What it is:** Scientific, metrics-based framework that produces quantifiable, repeatable security test results using Risk Assessment Values (RAVs).

**5 channels:** Human (HUMSEC), Physical (PHYSSEC), Wireless (SPECSEC), Telecom (COMSEC), Data Networks (DATASEC)

**Use it when:** Multiple assessors need to compare results, client wants to track improvement over time, or engagement requires defensible quantitative output.

**Why it matters:** Only major framework that covers the full attack surface including physical and human factors. A perfect firewall means nothing if someone can tailgate into the server room.

**Limitation:** Steep learning curve. Full implementation is time-consuming.

→ [[OSSTMM]]

---

### OWASP WSTG — Web Security Testing Guide
**What it is:** 90+ discrete test cases across 12 categories covering the full attack surface of modern web applications. Risk-based, continuously updated by global community.

**12 categories include:** Authentication, Authorization, Session Management, Input Validation, Business Logic, API Testing, and more.

**Use it when:** Web application penetration test. Supplement PTES with WSTG for the web-specific portion of any engagement.

**Why it matters:** The de facto standard for web app security testing. Covers modern architectures — SPAs, microservices, APIs. Each test case has step-by-step procedures.

**Limitation:** 90+ tests — full implementation is impractical for small teams. Risk of checklist mentality.

→ [[OWASP WSTG]]

---

### OWASP MASTG — Mobile Application Security Testing Guide
**What it is:** Mobile counterpart to WSTG. Detailed test cases for Android and iOS application security. Used alongside MASVS (which defines requirements; MASTG defines how to test them).

**Covers:** Data storage, cryptography, network communication, platform interaction, code quality.

**Use it when:** Any engagement involving a mobile application — banking apps, healthcare portals, consumer apps.

**Why it matters:** Only comprehensive mobile-specific testing framework. No substitute for mobile engagements.

→ [[Other Frameworks]]]

---

### NIST SP 800-115 — Technical Guide to Information Security Testing
**What it is:** Foundational framework from the National Institute of Standards and Technology covering the full spectrum of security testing — document reviews through full penetration tests.

**3 phases:** Planning → Execution (Review, Target ID, Vulnerability Validation, Penetration Testing) → Post-Testing

**Use it when:** U.S. government agencies, federal contractors, or any engagement where results must satisfy auditors and be defensible to oversight.

**Why it matters:** Institutional credibility. Immediate recognition in government, defense, and regulated industries. Flexible enough to adapt to cloud and IoT.

**Limitation:** Guidance only — not regulation. Doesn't mandate audit frequencies like PCI DSS does.

→ [[NIST SP 800-115]]

---

### ISSAF — Information Systems Security Assessment Framework
**What it is:** Legacy open-source pentest framework with a clear 9-step assessment model mirroring how real attackers progress through an environment.

**9 steps:** Information Gathering → Network Mapping → Vulnerability ID → Penetration → Privilege Escalation → Enumerate Further → Lateral Movement → Maintain Access → Cover Tracks

**Use it when:** Study it for adversarial logic and kill-chain thinking. Do NOT use for active engagements — tool guidance is 15+ years out of date.

**Why it matters:** The 9-step progression is one of the clearest mental models for how attacks unfold. Maps directly to kill-chain thinking.

→ [[ISSAF]]

---

### MITRE ATT&CK — Adversarial Tactics, Techniques, and Common Knowledge
**What it is:** A knowledge base of real-world adversary behavior organized as a matrix of 14 tactics and 200+ techniques with sub-techniques, built from observed threat actor activity.

**Use it when:** Enriching pentest reports by mapping findings to technique IDs. Threat intelligence. Detection engineering. Red teaming.

**Why it matters:** Universal translator of adversary behavior. Enables pentesters, threat intel, detection engineers, and IR teams to speak the same language. Shifts client conversation from "fix this bug" to "can we detect this class of attack."

**Key distinction:** Not a pentest methodology — a vocabulary for describing what you found.

→ [[MITRE ATTCK]]

---

### Cyber Kill Chain
**What it is:** Lockheed Martin's 7-stage framework describing how cyberattacks progress from reconnaissance to objectives.

**7 stages:** Reconnaissance → Weaponization → Delivery → Exploitation → Installation → Command & Control → Actions on Objectives

**Use it when:** Understanding attack progression, structuring threat analysis, teaching defenders where to interrupt an attack.

**Why it matters:** Defenders can break the attack at any stage. The earlier the detection, the less damage. Red teamers follow this chain to ensure realistic simulations.

→ [[Cyber Kill Chain]]

---

### PCI DSS Penetration Testing Guidelines
**What it is:** Regulatory mandate in PCI DSS v4.0 Requirement 11.4. Not optional for any organization processing, storing, or transmitting payment card data.

**Mandates:** Annual testing minimum, after significant infrastructure changes, covers external perimeter AND internal network, validates network segmentation controls.

**Use it when:** Retailer, payment processor, fintech, or any entity in the payment card ecosystem.

→ [[Other Frameworks]]

---

### CBEST
**What it is:** Threat-intelligence-led penetration testing framework developed by the Bank of England for UK financial institutions. Begins with a bespoke threat intelligence phase identifying relevant threat actors for that specific institution.

**Use it when:** UK banks, insurers, financial market infrastructure providers under Bank of England oversight.

→ [[Other Frameworks]]

---

### CSA Cloud Controls Matrix (CCM)
**What it is:** Governance and compliance framework for cloud environments mapping controls across 17 domains. Aligns with ISO 27001, NIST, and PCI DSS.

**Use it when:** Cloud security posture assessment — not a pentest methodology.

→ [[Other Frameworks]]

---

## Framework Selection Decision Tree

**Is the target a web application?**
→ OWASP WSTG (supplement with PTES for overall structure)

**Is the target a mobile app?**
→ OWASP MASTG (add PCI DSS guidelines if it handles card data)

**Is the client a UK financial institution?**
→ CBEST

**Does the client process payment card data?**
→ PCI DSS Guidelines (mandatory, not optional)

**Is the client a US government agency or federal contractor?**
→ NIST SP 800-115

**Does the client need year-over-year comparison or quantifiable results?**
→ OSSTMM

**Is it a standard corporate network pentest?**
→ PTES (primary) + MITRE ATT&CK (report enrichment)

**Is the scope multi-channel (physical, human, wireless, network)?**
→ OSSTMM

**Do you need to enrich a report with real-world threat context?**
→ MITRE ATT&CK (always, regardless of primary framework)

---

## The Combinations That Come Up Most

| Engagement Type | Primary | Supplement With |
|---|---|---|
| Corporate network pentest | PTES | MITRE ATT&CK |
| Web app pentest | OWASP WSTG | PTES (structure), MITRE ATT&CK (reporting) |
| Mobile app with payments | OWASP MASTG | PCI DSS Guidelines |
| Government assessment | NIST SP 800-115 | MITRE ATT&CK |
| Multi-year tracked assessment | OSSTMM | OWASP WSTG (if web) |
| UK bank | CBEST | MITRE ATT&CK |
