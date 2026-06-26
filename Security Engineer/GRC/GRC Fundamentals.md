# Governance, Risk Management & Compliance (GRC)

GRC is the framework that ties together how an organization directs its security strategy, manages risk, and ensures it meets legal and regulatory obligations. As a Network Security Engineer you'll work within GRC frameworks constantly — your controls, policies, and architecture decisions all feed into an organization's compliance posture.

---

## Core Terminology

| Term | Definition |
|---|---|
| **Governance** | Managing and directing an organization to achieve its objectives and ensure compliance with laws, regulations, and standards |
| **Regulation** | A rule or law enforced by a governing body to ensure compliance and protect against harm |
| **Compliance** | The state of adhering to laws, regulations, and standards that apply to an organization |
| **Policy** | A formal statement outlining an organization's goals, principles, and guidelines |
| **Standard** | A document establishing specific requirements for a particular process, product, or service |
| **Guideline** | Recommendations and best practices — non-mandatory |
| **Procedure** | Specific steps for undertaking a particular task or process |
| **Baseline** | Minimum security standards or requirements an organization must meet |

---

## Information Security Governance

Top-level management responsibility covering:

- **Strategy** — information security strategy aligned with business objectives
- **Policies and procedures** — governing use and protection of information assets
- **Risk management** — identify threats, implement mitigations
- **Performance measurement** — metrics and KPIs to measure effectiveness
- **Compliance** — ensuring adherence to regulations and industry standards

---

## The GRC Framework

Three integrated components — not separate silos:

### Governance Component
Sets direction through:
- Information security strategy
- Policies, standards, baselines, frameworks
- Performance monitoring and measurement

### Risk Management Component
- Identify, assess, and prioritize risks
- Implement controls and mitigation strategies
- Continuously monitor and refine

### Compliance Component
- Ensure legal, regulatory, and industry obligations are met
- Develop and implement compliance programs
- Conduct regular audits and assessments
- Report compliance issues to stakeholders

### Developing a GRC Program — Steps

1. **Define scope and objectives** — what's covered, what are the goals
2. **Conduct risk assessment** — identify and prioritize cyber risks
3. **Develop policies and procedures** — password policy, access control, etc.
4. **Establish governance processes** — steering committees, defined roles
5. **Implement controls** — firewalls, IDS/IPS, SIEM, training
6. **Monitor and measure performance** — track metrics, compliance
7. **Continuously improve** — review, adjust, post-incident analysis

---

## Information Security Frameworks — Document Types

| Document | Purpose | Mandatory? |
|---|---|---|
| Policy | High-level goals and principles | Yes (internally) |
| Standard | Specific requirements for a process/product | Yes |
| Guideline | Best practice recommendations | No |
| Procedure | Step-by-step task instructions | Yes (when invoked) |
| Baseline | Minimum security requirements | Yes |

### Developing Governance Documents — Process

1. Identify scope and purpose
2. Research relevant laws, regulations, standards
3. Draft the document
4. Review and approval by stakeholders
5. Implementation and communication
6. **Review and update** — ongoing, based on threat landscape changes

---

## Key Laws and Regulations

### GDPR — General Data Protection Regulation

- **Who:** EU regulation — applies to any organization handling EU citizens' personal data
- **What:** Strict requirements for collecting, storing, and processing personal data
- **Key principles:**
  - Prior consent required before collecting personal data
  - Data minimization — only collect what's necessary
  - Adequate protection measures required
  - Individuals have right to access, correct, and delete their data
- **Penalties:**
  - Tier 1 (severe violations — unauthorized data collection, sharing without consent): Up to **4% of annual revenue or €20 million**, whichever is higher
  - Tier 2 (less severe — breach notification failures, policy issues): Up to **2% of annual revenue or €10 million**, whichever is higher

### HIPAA — Health Insurance Portability and Accountability Act

- **Who:** US healthcare organizations and their business associates
- **Domain:** Healthcare
- **What:** Protects the privacy and security of health information (PHI — Protected Health Information)
- **Key requirements:** Access controls, audit logs, encryption, breach notification

### PCI-DSS — Payment Card Industry Data Security Standard

- **Who:** Any organization that handles payment card data (merchants, processors, service providers)
- **Domain:** Financial / Payment
- **CHD:** Cardholder Data — the data PCI-DSS protects
- **What:** Technical and operational requirements for secure card transactions
- **Key requirements:** Encryption, access control, network monitoring, web application firewalls, regular testing
- **Established by:** Visa, MasterCard, American Express, Discover

### GLBA — Gramm-Leach-Bliley Act

- **Who:** US financial companies
- **What:** Requires financial institutions to protect customers' nonpublic personal information (NPI)
- **Key requirements:** Information security programs, privacy notices, data sharing disclosure

### PIPEDA — Personal Information Protection and Electronic Documents Act

- **Who:** Canadian private sector organizations
- **What:** Governs how private sector organizations collect, use, and disclose personal information

---

## NIST Frameworks

### NIST SP 800-53

**"Security and Privacy Controls for Information Systems and Organizations"**

Developed by NIST (National Institute of Standards and Technology). Provides a comprehensive catalog of security and privacy controls organized into 20 control families. Widely used by US federal agencies and contractors, and adopted voluntarily by private sector.

**20 Control Families (key ones):**

| Control Family | Abbreviation |
|---|---|
| Access Control | AC |
| Awareness and Training | AT |
| Audit and Accountability | AU |
| Configuration Management | CM |
| Contingency Planning | CP |
| Identification and Authentication | IA |
| Incident Response | IR |
| Maintenance | MA |
| **Media Protection** | **MP** |
| Personnel Security | PS |
| Physical and Environmental Protection | PE |
| Planning | PL |
| Program Management | PM |
| Risk Assessment | RA |
| System and Services Acquisition | SA |
| System and Communications Protection | SC |
| System and Information Integrity | SI |

**Key control for exam questions:**
- Media Protection → **MP** family
- Incident Response → **IR** family

**Program Management (PM)** — one of the most important controls. Mandates establishing, implementing, and monitoring organization-wide security and privacy programs.

**Compliance best practices:**
1. Discovery — catalog data assets, systems, threats
2. **Mapping** — map 800-53 control families to identified assets (correlating assets and permissions)
3. Governance structure — allocate duties, implementation procedures
4. Monitor and evaluate — regular audits and assessments

### NIST SP 800-63B

Guidelines for **digital identity** and authentication. Covers:
- Identity assurance levels
- Authentication factors (passwords, biometrics, tokens)
- Secure credential management and storage

**Practical relevance:** When designing authentication systems, 800-63B tells you what's acceptable at each assurance level. This is why MFA requirements exist.

---

## ISO/IEC Standards

### ISO/IEC 27001

International standard for **Information Security Management Systems (ISMS)**. Internationally recognized, auditable, and certifiable.

**Core components:**

| Component | Description |
|---|---|
| Scope | Boundaries of the ISMS — what assets and processes are covered |
| Information Security Policy | High-level organizational security approach |
| Risk Assessment | Identify and evaluate risks to CIA |
| **Risk Treatment** | Select and implement controls to reduce identified risks to acceptable level |
| Statement of Applicability (SoA) | Which controls from the standard apply and which don't |
| Internal Audit | Periodic audits to verify ISMS is operating effectively |
| Management Review | Regular review of ISMS performance |

**Risk Treatment** is the component where controls are selected and implemented.

**ISO 27001 vs NIST 800-53:**
- ISO 27001 — internationally recognized, certifiable, framework-based
- NIST 800-53 — US government focused, more prescriptive control catalog
- Many organizations implement both or map between them

### ISO/IEC 19249

Architectural and design principles for secure products and systems. See [[security-principles-notes]] for the full breakdown.

---

## SOC 2 — Service Organization Control 2

Developed by AICPA (American Institute of Certified Public Accountants). Auditing framework focused on how service organizations protect customer data.

**Who needs it:** Cloud providers, SaaS companies, data processors — any third-party service that handles client data.

**Trust Service Criteria (what's evaluated):**
- **Security** — protection against unauthorized access
- **Availability** — system remains available for operation
- **Processing Integrity** — system processing is complete, accurate, timely
- **Confidentiality** — confidential information is protected
- **Privacy** — personal information is collected, used, and disclosed appropriately

**Generic controls checked in a SOC 2 audit:**
- Availability controls — system remains available (this is the "availability" criterion)
- Logical access controls
- Change management
- Risk management
- Incident response
- Physical security
- Data encryption in transit and at rest
- Network security controls

**SOC 2 audit process:**
1. Determine scope
2. Choose qualified auditor
3. Plan the audit
4. Prepare — review controls, identify gaps
5. Conduct the audit — interviews, documentation review, control testing
6. Receive audit report — findings and recommendations

**SOC 2 Type I vs Type II:**
- Type I — snapshot: controls are designed appropriately at a point in time
- Type II — over time: controls are operating effectively over a period (usually 6-12 months) — more valuable

---

## Compliance Frameworks by Industry

| Industry | Primary Frameworks |
|---|---|
| Healthcare | HIPAA, ISO 27001 |
| Financial Services | PCI-DSS, GLBA, SOC 2, ISO 27001 |
| US Federal Government | NIST 800-53, FedRAMP |
| EU Operations | GDPR, ISO 27001 |
| Cloud Providers | SOC 2, ISO 27001, FedRAMP |
| Any Organization | ISO 27001, NIST CSF |

---

## GRC in Practice — What This Means for a Network Security Engineer

You won't be writing policies or running audits day-to-day, but you'll be implementing the technical controls that satisfy compliance requirements. Examples:

| Compliance Requirement | Technical Control You Implement |
|---|---|
| NIST 800-53 — Audit and Accountability | Configure syslog, SIEM ingestion, log retention |
| PCI-DSS — Network segmentation | VLANs, firewall rules isolating cardholder data environment |
| ISO 27001 — Access control | AAA, RADIUS, role-based access on network devices |
| HIPAA — Encryption | TLS on all connections carrying PHI, encrypted storage |
| SOC 2 — Availability | Redundant links, failover configuration, uptime monitoring |
| GDPR — Data protection | Encryption, access controls, audit logging |

Understanding the framework helps you understand why the requirement exists, which makes you better at implementing it and talking about it in interviews.

---

## Key Definitions for Exam/Interview

| Term | One-liner |
|---|---|
| Regulation | Rule enforced by a governing body |
| Governance | Directing an organization to achieve objectives and ensure compliance |
| Compliance | Adhering to applicable laws, regulations, and standards |
| GRC | Integrated framework for Governance, Risk management, and Compliance |
| ISMS | Information Security Management System — the thing ISO 27001 certifies |
| PII | Personally Identifiable Information — what privacy regulations protect |
| PHI | Protected Health Information — what HIPAA protects |
| CHD | Cardholder Data — what PCI-DSS protects |
| NPI | Nonpublic Personal Information — what GLBA protects |
| Risk Treatment | Selecting and implementing controls to reduce risk to acceptable level |
| SoA | Statement of Applicability — ISO 27001 document listing applicable controls |

---

## Resources

- NIST 800-53 controls: https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final
- NIST 800-63B digital identity: https://pages.nist.gov/800-63-3/sp800-63b.html
- NIST Cybersecurity Framework: https://www.nist.gov/cyberframework
- ISO 27001 overview: https://www.iso.org/isoiec-27001-information-security.html
- GDPR full text: https://gdpr-info.eu
- PCI-DSS: https://www.pcisecuritystandards.org
- SOC 2 overview: https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/sorhome.html
- OWASP security cheat sheets: https://cheatsheetseries.owasp.org
