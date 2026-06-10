# AI and ML Security Threats

**THM Room:** AI and ML Security Threats  
**Path:** AI Security  
**Completed:** June 2026

---

## The AI Stack (How It All Connects)

| Term | What It Is |
|---|---|
| **AI** | Overarching field — machines mimicking human intelligence |
| **ML** | Subfield of AI — learns patterns from data without explicit instructions |
| **DL** | Subfield of ML — uses neural networks, self-learning, no human labeling needed |
| **LLMs** | Built on DL transformers — understand and generate human-like text |

---

## ML Algorithm Types

| Type | How It Works |
|---|---|
| Supervised | Labeled data — classification/regression (e.g. spam detection) |
| Unsupervised | Unlabeled data — finds hidden patterns via clustering |
| Semi-supervised | Mix of labeled and unlabeled data |
| Reinforcement | Learns by reward/penalty — agent refines actions over time |

---

## Neural Networks

- Nodes = neurons, connections = synapses
- **Input layer** — receives raw data
- **Hidden layers** — extract and refine features
- **Output layer** — produces final prediction
- Connections have **weights** — determines importance of each connection
- 3+ layers = Deep Learning

---

## LLMs — How They Work

1. **Pre-training** — processes massive datasets (GPT-3 = 2,600 years of reading)
2. Uses **transformer neural networks** (Google, 2017 — "Attention Is All You Need")
3. Assigns **attention scores** to words for contextual understanding
4. **Backpropagation** — fine-tunes parameters based on prediction accuracy
5. **RLHF** (Reinforcement Learning from Human Feedback) — humans review and correct outputs post-training

---

## AI Vulnerabilities (New Attack Surface)

| Vulnerability | Description |
|---|---|
| **Prompt Injection** | Overrides original system instructions to make the model behave maliciously or disclose sensitive info |
| **Data Poisoning** | Manipulates training data to corrupt model output (e.g. making spam filter fail) |
| **Model Theft** | Attacker queries model API, uses outputs to train a clone |
| **Privacy Leakage** | Model inadvertently reveals sensitive training data (e.g. patient records) |
| **Model Drift** | Model performance degrades over time as environment/data changes |

---

## AI-Enhanced Attacks

| Attack | How AI Enhances It |
|---|---|
| **Malware** | Generative AI generates malware code instantly with minimal effort |
| **Deepfakes** | Replicates voice/appearance with high accuracy — bypasses human authentication |
| **Phishing** | Generates fluent, context-aware phishing emails regardless of attacker's writing ability |

---

## AI for Defense

| Capability | Application |
|---|---|
| **Analysis** | ML detects anomalies in network traffic — intrusion detection at scale |
| **Prediction** | Automates security workflows — identifies phishing before it reaches users |
| **Summarization** | Digests incident reports, draws correlations between events |
| **Investigation** | Feed logs to LLM — identify attacks, get queries for triage, threat hunt |

**IBM Cost of a Data Breach findings:**
- Companies using AI saved avg **$2.2M** per breach
- AI identifies and contains breaches **108 days faster**
- Only **24%** of gen AI initiatives are currently secured

---

## Securing AI

- **Access controls** — RBAC and MFA to restrict who can interact with models
- **Encrypt training data** — treat it like any sensitive data
- **Follow standards** — ISO/IEC 27090 for AI security guidance
- **Monitor models** — detect unexpected behavior using explainability tools (SHAP, LIME)

---

## Key Framework

**MITRE ATLAS** — AI-specific extension of MITRE ATT&CK. Maps adversarial tactics and techniques targeting AI systems. Use alongside ATT&CK for full coverage.

---

## Useful Tools Referenced

- **Microsoft Defender for Endpoint** — AI-powered threat detection
- **Splunk** — AI-enhanced log analysis
- **SHAP / LIME** — Model explainability and monitoring tools

---

## Flag
`thm{443/60/16384}`
- DNS over HTTPS (DoH) = port 443
- SYN flood timeout = 60 seconds  
- Windows ephemeral port range size = 16384
