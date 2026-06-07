# ISSAF — Information Systems Security Assessment Framework

Developed by the Open Information Systems Security Group (OISSG). Open-source penetration testing framework. **Last version: v0.2.1, published ~2006. No longer actively maintained.**

**Why study it anyway:** The 9-step assessment model is one of the clearest representations of how an attacker progresses through a target environment. Mirrors kill-chain thinking.

**Use for:** Methodology and adversarial logic only — not tool guidance (outdated).

---

## The 3 Phases

### Phase 1 — Planning and Preparation
- Define scope and engagement boundaries
- Establish escalation protocols and emergency contacts
- Identify constraints (e.g. production systems off-limits during business hours)
- Agree on toolset

---

### Phase 2 — Assessment (The 9-Step Model)

| Step | Activity |
|---|---|
| 1 | **Information Gathering** — DNS, WHOIS, LinkedIn, job postings, tech stack recon |
| 2 | **Network Mapping** — map live topology, discover hosts and services |
| 3 | **Vulnerability Identification** — scan mapped assets for weaknesses |
| 4 | **Penetration** — initial exploitation attempt |
| 5 | **Gaining Access & Privilege Escalation** — escalate from initial foothold |
| 6 | **Enumerating Further** — with elevated access, discover what's now reachable |
| 7 | **Lateral Movement** — move to other systems using harvested credentials |
| 8 | **Maintaining Access** — demonstrate persistent access (document, don't necessarily deploy) |
| 9 | **Covering Tracks** — identify logging gaps, demonstrate how attacker would erase evidence |

**Progression:** Steps 1-3 = recon and analysis → Steps 4-7 = active compromise → Steps 8-9 = persistence and stealth

---

### Phase 3 — Reporting and Cleanup
- Structured report prioritized by business impact
- Remove all test artifacts
- Revoke temporary accounts
- Confirm no testing residue remains in environment

---

## Strengths and Weaknesses

| Strengths | Weaknesses |
|---|---|
| 9-step model clearly mirrors real attacker progression | No longer maintained |
| Excellent educational tool for understanding attack logic | Tool guidance is 15+ years out of date |
| Covers network, host, web, database, and social engineering | No active community |

---

## Key Takeaway
Study ISSAF for its adversarial logic and methodology. Supplement with PTES, OWASP WSTG, or current tool documentation for actual engagements.
