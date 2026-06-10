# PTES — Penetration Testing Execution Standard

Available at pentest-standard.org. Developed by experienced security practitioners to define what a real penetration test looks like end-to-end. Focuses on **how the engagement flows** from first client call to final report delivery.

**Best for:** Standard corporate network penetration tests. Most closely mirrors actual pentester workflow.

---

## The 7 Phases

### Phase 1 — Pre-Engagement Interactions
Everything before testing begins.

- Define scope with client
- Document rules of engagement
- Establish testing windows and emergency contacts
- Get written authorization ("get out of jail free" letter)
- **Most detailed phase in PTES** — unclear scoping is the #1 source of legal and professional disputes

---

### Phase 2 — Intelligence Gathering
Passive and active recon about the target.

**Passive:**
- Employee harvesting (LinkedIn)
- Subdomain discovery via certificate transparency logs
- Job postings revealing tech stack
- WHOIS, DNS records

**Active:**
- DNS enumeration
- Network scanning within agreed scope

---

### Phase 3 — Threat Modeling
Use gathered intel to identify highest-value targets and most likely attack paths.

- Prioritize assets by value
- Map primary attack paths
- Direct testing effort by adversarial logic, not random scanning

---

### Phase 4 — Vulnerability Analysis
Systematically identify weaknesses that enable the attack paths from threat model.

- Automated scanning + manual verification
- Eliminate false positives before exploitation

---

### Phase 5 — Exploitation
Attempt to exploit confirmed vulnerabilities.

- Goal: demonstrate **business impact**, not just pop boxes
- Exploitation should be purposeful and scoped

---

### Phase 6 — Post-Exploitation
Determine real-world impact of access gained.

- Pivot to other systems
- Extract credentials, sensitive data
- Demonstrate lateral movement
- Translate technical findings into business risk
- "We accessed 50,000 patient records" > "We got a shell"

---

### Phase 7 — Reporting
Deliver findings to two audiences:

| Audience | Content |
|---|---|
| Executive summary | Business risk in plain language, regulatory exposure, urgency |
| Technical report | Exact exploitation steps, affected hosts, evidence, prioritized remediation |

---

## Strengths and Weaknesses

| Strengths | Weaknesses |
|---|---|
| Practical end-to-end structure | Not formally updated in several years |
| Detailed pre-engagement guidance | Tool-specific guidance is outdated |
| Excellent for learning workflow instincts | Lacks quantitative metrics (relies on tester judgment) |

---

## Use PTES When
- Standard corporate network penetration test
- Client wants structured end-to-end methodology
- Need clear executive + technical reporting structure
- Supplement with MITRE ATT&CK to map findings to real-world threat behavior
