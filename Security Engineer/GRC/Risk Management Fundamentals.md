# Risk Management

Core GRC discipline. The R in GRC. Relevant for security engineering roles at the conceptual level — you need to speak this language with leadership and compliance teams, not run the math daily.

---

## Core Definitions

| Term | Definition |
|---|---|
| **Threat** | Intentional or accidental event that could compromise a system. Human-made, technical, or natural. |
| **Vulnerability** | Weakness in a system that a threat can exploit to cause harm. |
| **Asset** | Valuable resource an organization depends on — hardware, software, data, documentation. |
| **Risk** | Probability that a threat exploits a vulnerability and causes adverse business impact. Risk = Threat × Vulnerability × Asset. |
| **Risk Management** | Process of identifying, assessing, and mitigating risk to maintain acceptable levels. |

---

## Risk Management Process (NIST SP 800-30)

Four phases, runs as a cycle not a sequence:

**1. Frame Risk**
Establish context before making any decisions. Define:
- Risk Assumptions — what threats and vulnerabilities are we assuming exist?
- Risk Constraints — what limits our ability to respond (budget, resources)?
- Risk Tolerance — what level of risk is acceptable?
- Priorities and Trade-offs — what business functions are highest priority?

**2. Assess Risk**
Identify and analyze:
- What are the threats?
- What vulnerabilities exist?
- What is the impact if exploited?
- What is the likelihood of exploitation?

**3. Respond to Risk**
Four valid responses (see below). "Ignore risk" is never valid.

**4. Monitor Risk**
Ongoing — track effectiveness of controls, watch for changes, ensure compliance.

Monitoring focuses on:
- **Effectiveness** — is the control still working?
- **Change** — new systems, new business functions, new attack surface?
- **Compliance** — new laws, regulations, audit findings?

---

## Risk Responses

| Response | What It Means | Example |
|---|---|---|
| **Avoid** | Eliminate the activity causing the risk | Ban internet access on all machines |
| **Transfer / Share** | Move the risk to a third party | Buy cyber insurance |
| **Mitigate / Reduce** | Implement controls to lower likelihood or impact | Install antivirus, patch systems |
| **Accept** | Acknowledge the risk, take no action | Cost of countermeasure exceeds potential loss |

> **Note:** Accepting risk ≠ ignoring risk. Acceptance requires a conscious, documented decision with business justification.

---

## Risk Analysis Methods

### Qualitative
Uses descriptive ratings: High / Medium / Low or Red / Yellow / Green.
Maps probability against impact on a matrix.
Used when you don't have enough data for precise numbers, or for quick triage.

### Quantitative
Uses monetary values. More defensible for business cases and budget decisions.

**Key formulas:**

```
SLE = Asset Value × Exposure Factor (EF)
```
- **SLE** (Single Loss Expectancy) — dollar loss from one occurrence of the threat
- **Asset Value** — monetary value of the asset
- **EF** (Exposure Factor) — percentage of the asset lost if the threat is realized

```
ALE = SLE × ARO
```
- **ALE** (Annualized Loss Expectancy) — expected loss per year
- **ARO** (Annualized Rate of Occurrence) — how many times the threat occurs per year

```
Value of Safeguard = ALE (before) − ALE (after) − Annual Cost of Safeguard
```
- If positive → the control is financially justified
- If negative → the control costs more than the risk it prevents

**Example:**
- Laptop value: $10,000. Ransomware EF: 90%.
- SLE = $10,000 × 0.90 = $9,000
- ARO = 0.5 (once every two years)
- ALE (before) = $9,000 × 0.5 = $4,500
- Antivirus cost: $120/year. Reduces ARO to 0.02.
- ALE (after) = $9,000 × 0.02 = $180
- Value of safeguard = $4,500 − $180 − $120 = **$4,200** → justified

---

## Supply Chain Risk

Risk doesn't only come from your own systems. Suppliers introduce risk through:

- **Hardware** — hardware trojans embedded by a compromised supplier
- **Software** — malicious code in a vendor's product (SolarWinds attack model)
- **Services** — a compromised service provider exposes your data

Relevant for evaluating third-party vendors, cloud services, and managed security providers.

---

## Threat Categories

| Category | Examples |
|---|---|
| **Human-made** | Cyberattacks, terrorism, arson, industrial accidents |
| **Technical** | Power outages, hardware failure, data breaches, network vulnerabilities |
| **Natural** | Earthquakes, floods, hurricanes — location-dependent |

---

## Interview Context

- "Walk me through how you'd evaluate whether a security control is justified." → Use the Value of Safeguard formula conceptually. You don't need to run exact numbers, just explain the logic: does the cost of the control exceed the expected loss it prevents?
- "How do you prioritize risk?" → Risk responses are based on severity × likelihood × cost of countermeasure.
- The language (SLE, ALE, ARO) shows up in conversations with CISOs, auditors, and compliance teams. Being fluent in it signals you understand security as a business function, not just a technical one.
