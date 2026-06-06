# Additional Frameworks and Framework Selection

---

## Specialized Frameworks — Landscape Awareness

| Framework | Domain | Type | Maintained | Use When |
|---|---|---|---|---|
| WASC Threat Classification | Web applications | Threat taxonomy | No — superseded by OWASP | Legacy references only |
| CSA Cloud Controls Matrix (CCM) | Cloud environments | Governance/compliance | Yes | Cloud security posture assessments |
| OWASP MASTG | Mobile apps (Android/iOS) | Testing guide | Yes | Mobile app penetration testing |
| PCI DSS Pen Testing Guidelines | Payment card environments | Regulatory mandate | Yes (v4.0) | Any engagement involving cardholder data |
| CBEST | UK financial sector | Threat-intel-led pentest | Yes | UK financial institutions under Bank of England oversight |

---

## Framework Details

### CSA Cloud Controls Matrix (CCM)
- Maps controls across 17 domains including data security, IAM, infrastructure security
- Aligns with ISO 27001, NIST, PCI DSS
- Not a pentest methodology — governance and compliance tool

### OWASP MASTG
- Mobile counterpart to OWASP WSTG
- Covers data storage, cryptography, network communication, platform interaction, code quality
- Used alongside MASVS (Mobile Application Security Verification Standard) — MASVS defines requirements, MASTG defines how to test them

### PCI DSS Penetration Testing Guidelines
- Requirement 11.4 in PCI DSS v4.0
- **Mandatory** for any org processing, storing, or transmitting cardholder data
- Must cover external perimeter AND internal network
- Must be conducted **at least annually** and after significant infrastructure changes
- Must validate network segmentation controls

### CBEST
- Developed by Bank of England with UK financial sector
- Begins with bespoke threat intelligence phase identifying relevant threat actors for that specific institution
- Penetration test simulates those realistic threat scenarios
- Required for UK banks, insurers, financial market infrastructure providers

---

## Framework Selection Criteria

### 1. Engagement Scope and Target Type
| Target | Framework |
|---|---|
| Web application | OWASP WSTG |
| Mobile application | OWASP MASTG |
| Full network pentest | PTES or OSSTMM |
| Multi-channel (physical, human, network) | OSSTMM |

### 2. Regulatory and Compliance Requirements
| Regulation | Framework |
|---|---|
| Payment card data (PCI DSS) | PCI DSS Pen Testing Guidelines |
| UK financial institution | CBEST |
| US government / federal contractor | NIST SP 800-115 |

### 3. Need for Quantifiable Results
- Multiple assessors comparing results → **OSSTMM** (RAV metrics)
- Year-over-year improvement tracking → **OSSTMM**

### 4. Team Expertise and Resources
- Small team, standard corporate pentest → **PTES**
- Full OSSTMM requires deep familiarity with metrics system
- CBEST requires threat intelligence capabilities

---

## Real-World Scenario Decisions

| Scenario | Best Framework(s) |
|---|---|
| Hospital — web portal + internal network, HIPAA compliance | PTES (primary) + OWASP WSTG (web) + MITRE ATT&CK (reporting enrichment) |
| UK bank needing regulatory-compliant test with threat intel phase | CBEST |
| SaaS startup — two firms testing over two years, need to compare results | OSSTMM (quantitative comparison) + OWASP WSTG (web test cases) |
| Fintech — Android/iOS mobile banking apps handling credit card transactions | OWASP MASTG + PCI DSS Guidelines |

---

## Key Takeaway
Framework selection is rarely a single choice. Real engagements use:
- A **primary framework** for overall methodology
- **Supplementary frameworks** for specific regulatory, platform, or reporting requirements

Knowing when each applies is what separates methodical pentesters from tool runners.
