# Threat Modeling

Threat modeling is a structured approach to identifying, prioritizing, and addressing potential security threats before they're exploited. It's how security teams think proactively about attack surfaces rather than reactively after incidents. Shows up in interviews, certifications (GCIH, CISSP, Security+), and is directly relevant to how Network Security Engineers design defensive architecture.

---

## Core Concepts

### Threat vs Vulnerability vs Risk

| Term | Definition | Analogy |
|---|---|---|
| **Threat** | Any potential occurrence or actor that could exploit a vulnerability | Someone breaking into your house |
| **Vulnerability** | A weakness or flaw that could be exploited | Broken lock or open window |
| **Risk** | The likelihood of a threat exploiting a vulnerability × the impact | Probability of burglary based on neighborhood crime rate and lack of alarm |

Risk = Likelihood × Impact — same formula as in GRC, just applied at the technical level.

---

## High-Level Threat Modeling Process

Generic process that all frameworks build upon:

1. **Define the scope** — which systems, applications, and networks are in scope
2. **Asset identification** — diagram the architecture and dependencies, classify assets by sensitivity
3. **Identify threats** — cyber attacks, physical attacks, social engineering, insider threats
4. **Analyze vulnerabilities and prioritize risks** — assess existing controls, rank by likelihood and impact
5. **Develop and implement countermeasures** — access controls, patching, segmentation
6. **Monitor and evaluate** — continuously test effectiveness, track mitigated risks

---

## Attack Trees

A graphical representation of how an attacker could achieve a goal. Hierarchical structure:
- **Root node** = attacker's primary goal
- **Child nodes** = strategies to achieve that goal
- **Leaf nodes** = specific techniques or steps

**Attack paths** = chains of vulnerabilities an attacker exploits in sequence to reach the goal. Each link in the chain is a stepping stone.

**Why it matters:** Attack trees make abstract threats concrete and visualizable. Useful for communicating risk to non-technical stakeholders and for identifying where a single control can break multiple attack chains.

---

## Frameworks

### MITRE ATT&CK

**What it is:** A globally accessible knowledge base of adversary tactics, techniques, and procedures (TTPs) based on real-world observations.

**Structure:**
- **Tactics** — the adversary's high-level objective (e.g., Initial Access, Persistence, Exfiltration)
- **Techniques** — how they achieve the tactic (e.g., Exploit Public-Facing Application — T1190)
- **Sub-techniques** — more specific implementations of a technique

**Each technique page contains:**
- Description and explanation
- Procedure examples — real threat groups that have used it
- Mitigations — recommended controls
- Detections — how to identify it in your environment
- References

**Three matrices:**
| Matrix | Covers |
|---|---|
| Enterprise | Traditional enterprise networks — Windows, Linux, macOS, cloud |
| Mobile | Smartphones and tablets |
| ICS | Industrial Control Systems — power plants, water treatment, transportation |

**Use cases:**
- Map identified threats to known TTPs
- Assess which techniques a specific threat group uses against your industry
- Identify detection gaps in your environment
- Prioritize vulnerability remediation based on active exploitation

**ATT&CK Navigator:** Web-based tool to visualize and annotate the ATT&CK matrix. Can filter by:
- Threat group (e.g., APT28, Lazarus Group, FIN7)
- Platform (Windows, GCP, Office 365, IaaS)
- Tactic

Useful for: mapping what techniques apply to your environment, visualizing coverage gaps, communicating threat landscape to stakeholders.

**Key technique IDs to know:**
| Technique | ID | Tactic |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Initial Access |
| Exploitation for Privilege Escalation | T1068 | Privilege Escalation |
| Data from Cloud Storage | T1530 | Collection |
| Network Denial of Service | T1498 | Impact |
| Phishing | T1566 | Initial Access |
| Valid Accounts | T1078 | Defense Evasion / Persistence |
| OS Credential Dumping | T1003 | Credential Access |

---

### DREAD Framework

**What it is:** A risk scoring model developed by Microsoft. Assigns numerical scores to vulnerabilities to prioritize them. Opinion-based — relies on analyst judgment, so consistency matters.

**Components:**

| Letter | Component | Question |
|---|---|---|
| D | **Damage** | How bad would an attack be? |
| R | **Reproducibility** | How easy is it to reproduce the attack? |
| E | **Exploitability** | How much work is it to launch the attack? |
| A | **Affected Users** | How many people will be impacted? |
| D | **Discoverability** | How easy is it to find the vulnerability? |

**Scoring:** Each component rated 1-10. Overall DREAD score = average of all five.

**Example scores:**
```
Unauthenticated RCE:           8.0  (D:10, R:7.5, E:10, A:10, D:2.5)
IDOR in User Profiles:         6.5  (D:2.5, R:7.5, E:7.5, A:10, D:5)
Server Misconfiguration:       5.0  (D:0, R:10, E:10, A:0, D:5)
```

**Best practices:**
- Establish consistent scoring guidelines with examples before scoring
- Score collaboratively — multiple teams reduce subjectivity
- Use alongside other frameworks, not in isolation

**When to use:** Quick prioritization of a list of vulnerabilities. Good for communicating risk in numerical terms to management.

---

### STRIDE Framework

**What it is:** A threat categorization methodology developed by Microsoft. Maps threats to the CIA triad and helps systematically identify threats in software systems.

**Components:**

| Letter | Threat | Policy Violated | Example |
|---|---|---|---|
| S | **Spoofing** | Authentication | Sending email as another user, phishing site |
| T | **Tampering** | Integrity | Modifying another user's password, installing backdoors |
| R | **Repudiation** | Non-repudiation | Denying a transaction when audit logs are missing |
| I | **Information Disclosure** | Confidentiality | Misconfigured database, public S3 bucket with sensitive data |
| D | **Denial of Service** | Availability | DDoS, ransomware encrypting shared resources |
| E | **Elevation of Privilege** | Authorization | Accessing admin console as regular user, exploiting unpatched system |

**STRIDE checklist approach** — map scenarios to which STRIDE categories apply:

| Scenario | S | T | R | I | D | E |
|---|---|---|---|---|---|---|
| Spoofed email, no logging | ✔ | | ✔ | | | |
| DDoS with no load balancing | | | | | ✔ | |
| SQL injection | | ✔ | | ✔ | | |
| Public S3 bucket with customer data | | | | ✔ | ✔ | |
| Local privilege escalation + backdoor | | ✔ | | | | ✔ |

**Process:**
1. System decomposition — break into components, identify trust boundaries
2. Apply STRIDE categories to each component
3. Threat assessment — impact and likelihood
4. Develop countermeasures per STRIDE category
5. Validate effectiveness
6. Continuous improvement

**When to use:** Software/application design and review. Excellent for catching design-level security flaws early in development. Pairs well with PASTA.

---

### PASTA Framework

**What it is:** Process for Attack Simulation and Threat Analysis. Risk-centric, seven-step framework that aligns security with business objectives. Created by Tony UcedaVélez and Marco Morana (2015).

**Seven steps:**

| Step | Name | What happens |
|---|---|---|
| 1 | **Define the Objectives** | Establish scope, security objectives, compliance requirements |
| 2 | **Define the Technical Scope** | Inventory assets — hardware, software, data. Understand architecture and data flows |
| 3 | **Decompose the Application** | Break system into components. Identify entry points, trust boundaries, attack surfaces, user roles |
| 4 | **Analyze the Threats** | Identify threats from all sources — external, insider, accidental. Use threat intel feeds |
| 5 | **Vulnerabilities and Weaknesses Analysis** | Identify exploitable weaknesses — misconfigs, bugs, unpatched systems. Use scanning and pen testing |
| 6 | **Analyze the Attacks** | Simulate attack scenarios. Build attack trees. Evaluate likelihood and impact |
| 7 | **Risk and Impact Analysis** | Implement countermeasures aligned with risk tolerance. Prioritize by severity |

**When to use:** Comprehensive organizational threat modeling, especially when business context matters. Most holistic of the four frameworks. Takes more time but produces more thorough output.

---

## Framework Comparison

| Framework | Best For | Approach | Output |
|---|---|---|---|
| **MITRE ATT&CK** | Mapping real adversary behavior, detection gap analysis | Tactics and techniques library | Heatmap of relevant techniques |
| **DREAD** | Quick numerical risk prioritization | Scored rating model | Ranked vulnerability list |
| **STRIDE** | Software/application threat identification | Categorical threat checklist | Threat checklist per component |
| **PASTA** | Comprehensive risk-centric organizational analysis | Seven-step methodology | Full risk assessment aligned to business |

---

## Teams Involved in Threat Modeling

| Team | Role |
|---|---|
| Security Team | Lead the exercise, expertise on threats and mitigations |
| Development Team | Ensure security in the software being built |
| IT and Operations | Network infrastructure, system config, integration knowledge |
| GRC Team | Align with compliance requirements and risk management objectives |
| Business Stakeholders | Identify critical assets, risk tolerance, business priorities |
| End Users | Identify user-specific vulnerabilities and interaction risks |

---

## How This Applies to Network Security Engineering

Threat modeling isn't just a GRC exercise — it directly informs how you design networks:

| Threat Modeling Output | Network Security Engineering Response |
|---|---|
| Identified attack path through public-facing app | WAF, network segmentation, IDS rules |
| STRIDE Denial of Service identified | Redundant links, rate limiting, DDoS protection |
| MITRE ATT&CK lateral movement techniques | Microsegmentation, zero trust architecture |
| STRIDE Elevation of Privilege | Least privilege on network devices, AAA controls |
| PASTA identifies unpatched systems as high risk | Automated patch management, vulnerability scanning |
| Attack tree shows single point of failure | Redundancy, failover design |

Understanding threat modeling makes you a better security architect because you design with attack paths in mind, not just compliance checkboxes.

---

## Quick Reference — Interview/Exam

| Framework | Key phrases |
|---|---|
| MITRE ATT&CK | Tactics, Techniques, real adversary behavior, ATT&CK Navigator |
| DREAD | Damage, Reproducibility, Exploitability, Affected Users, Discoverability — score 1-10, average |
| STRIDE | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege — built on CIA triad |
| PASTA | Seven steps, risk-centric, aligns to business objectives, attack simulation |
| Attack tree | Root = goal, branches = strategies, leaves = techniques |
| Threat vs Vulnerability vs Risk | Threat exploits vulnerability to create risk |

---

## Resources

- MITRE ATT&CK: https://attack.mitre.org
- ATT&CK Navigator: https://mitre-attack.github.io/attack-navigator
- MITRE ATLAS (AI/ML threats): https://atlas.mitre.org
- PASTA book: "Risk Centric Threat Modeling" — UcedaVélez and Morana
- Threat modeling cheat sheet (OWASP): https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
