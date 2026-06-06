# MITRE ATT&CK

**ATT&CK = Adversarial Tactics, Techniques, and Common Knowledge**

Developed and maintained by MITRE Corporation. Not a penetration testing framework — a **knowledge base of adversary behavior** built from real-world observations. Catalogs what attackers actually do.

**Key distinction:** ATT&CK doesn't tell you how to run a pentest. It gives you a standardized vocabulary for describing what you found.

---

## The Analogy
- **PTES** = diagnostic procedure (how to run the test)
- **ATT&CK** = medical dictionary (standardized terms for what you observe)

You need both — they serve different purposes.

---

## Structure: The Matrix

**Columns = Tactics** (the adversary's high-level objective — the *why*)
**Rows = Techniques** (specific methods to achieve that objective — the *how*)
**Sub-techniques** = further breakdown of techniques

### The 14 Enterprise Tactics (in order)
1. Initial Access
2. Execution
3. Persistence
4. Privilege Escalation
5. Defense Evasion
6. Credential Access
7. Discovery
8. Lateral Movement
9. Collection
10. Command and Control
11. Exfiltration
12. Impact

---

## Technique IDs
Format: `T[number]` for techniques, `T[number].[sub]` for sub-techniques.

| Example | Tactic | Description |
|---|---|---|
| T1566 | Initial Access | Phishing |
| T1566.001 | Initial Access | Spearphishing Attachment |
| T1566.002 | Initial Access | Spearphishing Link |
| T1190 | Initial Access | Exploit Public-Facing Application |
| T1078 | Initial Access | Valid Accounts |
| T1003 | Credential Access | OS Credential Dumping |
| T1550 | Lateral Movement | Use Alternate Authentication Material |
| T1213 | Collection | Data from Information Repositories |

---

## How to Use ATT&CK in a Pentest Report
Map findings to ATT&CK technique IDs to enrich reporting:

| Finding | Tactic | Technique |
|---|---|---|
| Phishing email delivered payload | Initial Access | T1566.001 |
| Exploited web app vulnerability | Initial Access | T1190 |
| Dumped cached credentials | Credential Access | T1003 |
| Moved laterally with stolen creds | Lateral Movement | T1550 |
| Accessed sensitive database | Collection | T1213 |

**Why this matters:** Client can look up each technique in ATT&CK, review detection guidance, and build detection rules. Shifts conversation from "fix this bug" to "can we detect this class of adversary behavior?"

---

## Each Technique Entry Includes
- Description
- Real-world threat groups that have used it
- Detection recommendations
- Mitigations

---

## Strengths
- Universal translator of adversary behavior
- Common language across pentesters, threat intel, detection engineers, and IR teams
- Grounded in real-world observed behavior
- Continuously updated

---

## Key Takeaway
ATT&CK complements your pentest framework by providing standardized vocabulary for findings. Over 200 techniques in the Enterprise matrix — mastery takes time but basic mapping is immediately useful in reports.
