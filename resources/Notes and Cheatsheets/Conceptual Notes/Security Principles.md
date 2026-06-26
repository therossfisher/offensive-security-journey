# Security Principles

Foundational concepts that underpin all of information security — offensive, defensive, GRC, architecture, and compliance. These frameworks show up in certifications (GSEC, GCIH, CISSP, Security+), job interviews, and real-world security design decisions.

---

## The CIA Triad

The core framework for evaluating the security of any system, data, or service.

| Principle | Definition | Attack Against It |
|---|---|---|
| **Confidentiality** | Only authorized parties can access data | Disclosure — unauthorized access, data breach, eavesdropping |
| **Integrity** | Data cannot be altered without detection | Alteration — tampering, man-in-the-middle, corruption |
| **Availability** | Systems and data are accessible when needed | Destruction/Denial — DDoS, ransomware, hardware failure |

**The opposite is DAD:**
- **D**isclosure — breach of confidentiality
- **A**lteration — breach of integrity
- **D**estruction/Denial — breach of availability

**Real-world tension:** These three goals conflict with each other. Maximum confidentiality and integrity (encrypt everything, restrict all access) reduces availability. Maximum availability (everything open, no authentication) destroys confidentiality and integrity. Security is about finding the right balance for the specific context.

**Examples by context:**

| Scenario | Primary Concern |
|---|---|
| Medical records | All three — confidentiality (HIPAA), integrity (wrong treatment = death), availability (emergency access) |
| University announcements | Integrity and availability — not confidential |
| Financial transactions | Integrity and confidentiality |
| Emergency services | Availability is paramount |
| Military communications | Confidentiality is paramount |

---

## Beyond CIA — Additional Properties

| Property | Definition |
|---|---|
| **Authenticity** | Data/document is genuinely from the claimed source — not forged or counterfeit |
| **Non-repudiation** | The source cannot deny having created or sent the data — legally binding trail |

**Why non-repudiation matters:** In banking, healthcare, and legal contexts, it's not enough to know data is authentic — you need proof the originating party can't deny. Digital signatures provide non-repudiation. Logs with integrity controls provide it for system actions.

---

## Parkerian Hexad

Donn Parker's 1998 expansion of CIA to six elements:

| Element | Definition |
|---|---|
| Confidentiality | Only authorized access |
| Integrity | Data accuracy and completeness |
| Availability | Accessible when needed |
| Authenticity | Genuinely from claimed source |
| **Utility** | Information is in a usable form — encrypted data you've lost the key to is available but has no utility |
| **Possession** | You maintain control of the data — ransomware attacks possession even if data isn't disclosed |

---

## Security Models

Formal mathematical models that define how security policies should be enforced in systems. You'll see these in certification exams and referenced in security architecture discussions.

### Bell-LaPadula Model — Confidentiality

Designed for military/government classification systems. Focuses on preventing unauthorized disclosure.

**Rules:**
- **No Read Up** (Simple Security Property) — a subject cannot read data at a higher classification level than their own
- **No Write Down** (Star Security Property) — a subject cannot write data to a lower classification level
- **Summary: "Write Up, Read Down"** — you can share up the chain, receive from below

**Limitation:** Not designed for file sharing or collaborative environments. Doesn't address integrity.

### Biba Model — Integrity

The integrity counterpart to Bell-LaPadula.

**Rules:**
- **No Read Down** (Simple Integrity Property) — a higher integrity subject should not read lower integrity data (prevents contamination)
- **No Write Up** (Star Integrity Property) — a lower integrity subject cannot write to higher integrity objects
- **Summary: "Read Up, Write Down"** — opposite of Bell-LaPadula

**Limitation:** Doesn't address insider threats. Doesn't handle confidentiality.

### Clark-Wilson Model — Integrity

More practical integrity model designed for commercial environments.

**Key concepts:**
- **CDI** (Constrained Data Item) — data whose integrity must be preserved
- **UDI** (Unconstrained Data Item) — user input, external data
- **TP** (Transformation Procedure) — authorized operations that maintain CDI integrity
- **IVP** (Integrity Verification Procedure) — checks that CDIs are valid

**Real-world analog:** Double-entry bookkeeping — every transaction must balance, every change is logged, and the integrity of financial records is verified by separate procedures.

### Other Security Models (Reference)
- Brewer and Nash (Chinese Wall) — prevents conflicts of interest
- Goguen-Meseguer — noninterference model
- Bell-LaPadula, Biba, Clark-Wilson — the three you'll see most often

---

## Security Principles

### Defence in Depth (Multi-Level Security)

No single control is sufficient. Layer multiple independent security controls so that failure of one doesn't compromise the whole system.

**Examples:**
- Network: Firewall → IDS/IPS → Network segmentation → Host firewall → Application controls
- Physical: Gate → Building lock → Floor access card → Room lock → Cable lock
- Data: Encryption at rest → Encryption in transit → Access controls → Audit logging

**Why it matters offensively:** As a pentester, defence in depth means you need to chain multiple vulnerabilities to achieve your objective. One unpatched service isn't enough if the network is segmented and the host has EDR.

---

### Zero Trust

**Core principle:** Never trust, always verify. Treat every entity — user, device, service, network location — as potentially adversarial until proven otherwise.

**Contrast with legacy models:** Traditional perimeter security trusted everything inside the network. Zero Trust eliminates the concept of a trusted internal network.

**Key requirements:**
- Every access request requires authentication and authorization regardless of network location
- Least privilege for every session
- Continuous verification — trust isn't permanent
- Microsegmentation — network segments as small as individual hosts, with authentication between them
- Assume breach — design as if attackers are already inside

**Real-world relevance:** Zero Trust is now the dominant enterprise security architecture model. Cloud, remote work, and supply chain attacks (SolarWinds, etc.) killed the perimeter model. If you're targeting enterprise security roles, understanding Zero Trust architecture is essential.

---

### Trust but Verify

Slightly less strict than Zero Trust. Trust is extended but verified through monitoring and logging.

**In practice:** You trust your employees — but you log their actions, review anomalies, and use SIEM to detect policy violations. The verification happens after the fact rather than as a prerequisite.

**Tooling:** SIEM, DLP, proxy, IDS/IPS, audit logs, user behavior analytics (UBA).

---

### Least Privilege

Give users, processes, and systems the minimum permissions necessary to perform their function — and nothing more.

**Examples:**
- A web server process should not run as root
- A read-only reporting user should not have write access to the database
- A developer shouldn't have production deployment rights unless required
- A service account should only access the specific resources it needs

**Security relevance:** Limiting blast radius. If a process or account is compromised, least privilege limits what the attacker can do. This is why post-exploitation privilege escalation is a necessary step — compromising a low-privilege account doesn't give full access.

---

### Attack Surface Minimization

Every enabled service, open port, installed application, and user account is potential attack surface. Reduce what's exposed.

**In practice:**
- Disable unused services and ports
- Remove unnecessary software
- Close unused accounts
- Restrict network access to required paths only
- Apply firewall rules to whitelist rather than blacklist

**Offensive relevance:** Attack surface enumeration is exactly what nmap, gobuster, and recon phases are doing — finding what's exposed. Defenders minimize surface; attackers find what wasn't minimized.

---

### Separation of Duties

No single person should have complete control over a critical process. Requiring multiple people to complete sensitive operations prevents fraud and limits insider threat.

**Examples:**
- Financial approval requires two signatures
- Code deployment requires separate developer and reviewer
- Backup creation and restoration tested by different people
- Admin access requires two-person authorization for critical systems

---

## ISO/IEC 19249 — Architectural and Design Principles

International standard for secure system design. Five architectural and five design principles.

### Architectural Principles

| Principle | Description | Example |
|---|---|---|
| **Domain Separation** | Group related components with shared security attributes into isolated domains | OS kernel in ring 0, user apps in ring 3; network segmentation by function |
| **Layering** | Structure systems in abstract layers, apply security at each layer | OSI model; application → transport → network → physical security controls |
| **Encapsulation** | Hide internal implementation details, expose only controlled interfaces | APIs instead of direct database access; OOP data hiding |
| **Redundancy** | Duplicate critical components to ensure availability and integrity | RAID, redundant power supplies, multi-region cloud deployment |
| **Virtualization** | Share hardware across isolated environments | VMs for malware analysis sandboxing; cloud isolation |

### Design Principles

| # | Principle | Description | Example |
|---|---|---|---|
| 1 | **Least Privilege** | Minimum permissions for the task | Read-only access for reporting users |
| 2 | **Attack Surface Minimization** | Disable everything not required | Turn off unused services |
| 3 | **Centralized Parameter Validation** | Validate all input in one place | Single input sanitization library for a web app |
| 4 | **Centralized Security Services** | Single point for security functions | Central authentication server (LDAP/AD) |
| 5 | **Prepare for Error and Exception Handling** | Fail safely; don't leak info in errors | Firewall blocks all on crash; generic error messages to users |

**Remember for exams:** These are numbered 1-5. Questions often give a scenario and ask which principle number applies.

---

## Vulnerability, Threat, and Risk

These three terms are often used interchangeably — incorrectly.

| Term | Definition | Example |
|---|---|---|
| **Vulnerability** | A weakness in a system | Unpatched Apache server with known CVE |
| **Threat** | A potential danger that could exploit the vulnerability | Attackers scanning for that CVE and running exploit code |
| **Risk** | The likelihood of the threat exploiting the vulnerability × the impact | High if server is internet-facing with sensitive data; lower if isolated internal system |

**Risk formula:** `Risk = Likelihood × Impact`

**Risk management options:**
- **Accept** — risk is low enough, do nothing
- **Mitigate** — implement controls to reduce likelihood or impact
- **Transfer** — cyber insurance, third-party contracts
- **Avoid** — eliminate the vulnerability entirely (remove the service)

---

## Shared Responsibility Model (Cloud)

In cloud environments, security responsibility is divided between the provider and the customer depending on the service model.

| Layer | IaaS | PaaS | SaaS |
|---|---|---|---|
| Data | Customer | Customer | Customer |
| Application | Customer | Customer | Provider |
| Runtime | Customer | Provider | Provider |
| OS | Customer | Provider | Provider |
| Virtualization | Provider | Provider | Provider |
| Hardware/Network | Provider | Provider | Provider |

**Key point:** Even in SaaS, the customer is still responsible for their data and user access management. Misconfigured S3 buckets are an IaaS customer responsibility — not AWS's.

---

## How These Concepts Show Up in Real Work

### In Penetration Testing
- CIA helps frame impact when writing findings — what specifically is at risk (confidentiality, integrity, availability)?
- Attack surface minimization tells you what a well-hardened target looks like vs. a poorly managed one
- Least privilege violations are findings — services running as root, overprivileged accounts

### In Security Engineering / Network Security
- Defence in depth drives architecture decisions — how many layers of control between the internet and critical systems?
- Zero Trust architecture design — microsegmentation, identity-based access
- Shared responsibility in cloud deployments

### In Incident Response
- CIA tells you what was impacted — data disclosed? altered? systems unavailable?
- DAD framework maps to the type of attack
- Non-repudiation and logging tell you whether you can prove what happened

### In GRC / Compliance
- CIA, Bell-LaPadula, and ISO 19249 principles map directly to compliance frameworks (NIST, ISO 27001, SOC 2)
- Risk = Likelihood × Impact is the foundation of every risk assessment

---

## Quick Reference — Exam Answers

| Concept | Key Phrase |
|---|---|
| Bell-LaPadula | No read up, no write down — confidentiality |
| Biba | No read down, no write up — integrity |
| Zero Trust | Never trust, always verify |
| Defence in Depth | Multiple independent layers of security |
| Least Privilege | Minimum permissions to perform the task |
| Non-repudiation | Cannot deny being the source |
| Authenticity | Genuinely from the claimed source |
| Utility (Parkerian) | Data is in a usable form |
| Possession (Parkerian) | Maintaining control of your data |
| Risk | Likelihood × Impact |
