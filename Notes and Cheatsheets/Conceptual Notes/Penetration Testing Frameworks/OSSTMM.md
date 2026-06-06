# OSSTMM — Open Source Security Testing Methodology Manual

Developed by ISECOM. A scientific, metrics-based approach to security testing. Goal: quantifiable, verifiable, repeatable results instead of subjective opinions.

**Current version:** 3
**Deliverable format:** STAR (Security Test Audit Report)

---

## Core Philosophy
Metrics over opinions. Two testers assessing the same target should produce comparable results — like engineers measuring the same structure.

---

## The 5 Security Channels

| Channel | Abbreviation | Covers |
|---|---|---|
| Human Security | HUMSEC | Social engineering, human-factor vulnerabilities |
| Physical Security | PHYSSEC | Badge readers, tailgating, physical access controls |
| Wireless Communications | SPECSEC | WiFi, Bluetooth, RFID, electromagnetic signals |
| Telecommunications | COMSEC | Phone systems, VoIP, fax, modems |
| Data Networks | DATASEC | Network services, firewalls, application-layer protocols |

**Why all 5 matter:** Perfect firewall rules mean nothing if an attacker can tailgate into the server room or social-engineer a password reset.

---

## Risk Assessment Values (RAVs)
OSSTMM's quantitative scoring system.

- Measures balance between **attack surface (exposure)** and **controls protecting it**
- **Positive RAV** = residual risk exists
- **RAV near zero** = controls are well-matched to exposure

---

## The 4 Testing Phases

### Phase 1 — Induction
**Enumeration and verification** — map what exists and confirm it's real.
- DNS queries, certificate transparency logs, subdomain discovery
- Verify each asset is live and responsive
- Output: confirmed inventory of target environment

### Phase 2 — Interaction
**Qualification and quantification** — actively probe and measure exposure.
- Connect to services, fingerprint technology
- Quantify attack surface (e.g. 12 reachable services, 4 accepting unauthenticated connections)
- Feeds into attack surface calculation

### Phase 3 — Inquiry
**Privilege escalation and verification escalation** — test whether exposure converts to unauthorized access.
- Exploit discovered vulnerabilities
- Verify and scope the impact (e.g. IDOR giving read access to 12,000 accounts)

### Phase 4 — Intervention
**Quarantine, audit, and enticement** — address findings and examine broader controls.
- **Quarantine** — restrict vulnerable endpoint while patch is developed
- **Audit** — examine wider control model for similar flaws
- **Enticement** — deploy canary tokens to test internal detection capabilities

---

## Strengths and Weaknesses

| Strengths | Weaknesses |
|---|---|
| Results are auditable and comparable | Steep learning curve |
| Scientific rigor | Time-consuming full implementations |
| Covers more than just network security | Fewer trained practitioners than OWASP/NIST |

**Best suited for:** Organizations needing repeatable, auditable security measurements willing to invest in the methodology.

---

## Compared to Other Frameworks
- **OWASP** — focused on web application security
- **NIST** — broader cybersecurity framework, policy-oriented
- **OSSTMM** — scientific, quantitative, covers all 5 channels including physical and human
